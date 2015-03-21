# Sets Functions {#sets}

1. [sadd](#sadd) - Adds one or more members to a set
1. [scard](#scard) - Gets the number of members in a set
1. [sdiff](#sdiff) - Subtracts multiple sets
1. [sdiffstore](#sdiffstore) - Subtracts multiple sets and store the resulting set in a key
1. [sinter](#sinter) - Intersects multiple sets
1. [sinterstore](#sinterstore) - Intersects multiple sets and store the resulting set in a key
1. [sismember](#sismember) - Determines if a given value is a member of a set
1. [smembers](#smembers) - Gets all the members in a set
1. [smove](#smove) - Moves a member from one set to another
1. [spop](#spop) - Removes and returns a random member from a set
1. [sscan](#sscan) - Iterates elements of Sets types
1. [srandmember](#srandmember) - Gets one or multiple random members from a set
1. [srem](#srem) - Removes one or more members from a set
1. [sunion](#sunion) - Adds multiple sets
1. [sunionstore](#sunionstore) - Adds multiple sets and store the resulting set in a key

### srandmember {#srandmember}
_**Description**_: Get one or multiple random members from a set, (not remove it).
See [Redis site](http://redis.io/commands/srandmember) for the use of additional parameters.

_**Parameters**_   
*number*: connection  
*string*: key name  
*(optional) number*: `count` distinct elements if count is positive. If count is negative, the number of elements is the absolute value of the specified count and can obtain the same element multiple times in the result  
*(optional) array*: containing results, when count parameter is used.

_**Return value**_   
*string*: the randomly selected element, or `null string` if key not exist. If count is used, returns `1` and the array parameter, will contain the results. 

{title="Example: Using srandmember",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      redis_del(c,"myset")
      A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
      redis_sadd(c,"myset",A)
      r=redis_smembers(c,"myset",MEMB)
      if(r!=-1) {
         print "Members in set 'myset'"
         for( i in MEMB) {
            print MEMB[i]
         }
      }
      print "srandmember gets: "redis_srandmember(c,"myset")
      print "Members in set 'myset', after appliying 'srandmember' function"
      delete MEMB
      r=redis_smembers(c,"myset",MEMB)
      print "smembers returns: "r
      for( i in MEMB) {
            print MEMB[i]
      }
      r=redis_srandmember(c,"myset",3,B)
       # Members obtained using srandmember with the additional count argument
      for( i in B) {
          print "              "B[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    Members in set 'myset'
    55
    c15
    89
    c16
    srandmember gets: c16
    Members in set 'myset', after appliying 'srandmember' function
    smembers returns: 1
    55
    c15
    89
    c16
              55
              c16
              c15

### spop {#spop}
_**Description**_: Removes and returns a random member from a set

_**Parameters**_   
*number*: connection  
*string*: key name  

_**Return value**_   
*string*: the removed element, or `null string` if key not exist.

{title="Example: Using spop",lang=text,linenos=off}
    BEGIN {
      c=redis_connect()
      redis_del(c,"myset")
      A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
      redis_sadd(c,"myset",A)
      r=redis_smembers(c,"myset",MEMB)
      if(r!=-1) {
         print "Members in set 'myset'"
         for( i in MEMB) {
            print MEMB[i]
         }
      }
      print "spop gets: "redis_spop(c,"myset")
      print "Members in set 'myset', after appliying 'spop' function"
      delete MEMB
      r=redis_smembers(c,"myset",MEMB)
      print "smembers returns: "r
      for( i in MEMB) {
            print MEMB[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    Members in set 'myset'
    55
    89
    c15
    c16
    spop gets: 55
    Members in set 'myset', after appliying 'spop' function
    smembers returns: 1
    89
    c15
    c16

### sdiff {#sdiff}
_**Description**_: Subtract multiple sets 

_**Parameters**_   
*number*: connection  
*array*: containing set names  
*array*: containing the members (strings) of the result

_**Return value**_   
*number*: `1` on sucess, `-1` on error.

{title="Example: Using sdiff",lang=text,linenos=off}
    redis_del(c,"myset1")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
    redis_sadd(c,"myset1",A)
    redis_del(c,"myset2")
    delete A
    A[1]="89"
    redis_sadd(c,"myset2",A)
    redis_del(c,"myset3")
    delete A
    A[1]="9"; A[2]="c16"; A[3]="89"
    redis_sadd(c,"myset3",A)
    delete A
    A[1]="myset1"; A[2]="myset2"; A[3]="myset3"
    redis_sdiff(c,A,RE)  # members expeced in array RE: 55, c15

### sinter {#sinter}
_**Description**_: Obtains the intersection of the given sets.

_**Parameters**_   
*number*: connection  
*array*: containing set names  
*array*: containing the members (strings) of the result

_**Return value**_   
*number*: `1` on sucess, `-1` on error.

{title="Example: Using sinter",lang=text,linenos=off}
    redis_del(c,"myset1")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
    redis_sadd(c,"myset1",A)
    redis_del(c,"myset2")
    delete A
    A[1]="89"
    redis_sadd(c,"myset2",A)
    redis_del(c,"myset3")
    delete A
    A[1]="9"; A[2]="c16"; A[3]="89"
    redis_sadd(c,"myset3",A)
    delete A
    A[1]="myset1"; A[2]="myset2"; A[3]="myset3"
    redis_sinter(c,A,RE)  # members expeced in array RE: 89

### sunion {#sunion}
_**Description**_: Obtains the union of the given sets.

_**Parameters**_   
*number*: connection  
*array*: containing set names  
*array*: containing the members (strings) of the result

_**Return value**_   
*number*: `1` on sucess, `-1` on error.

{title="Example: Using sunion",lang=text,linenos=off}
    redis_del(c,"myset1")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
    redis_sadd(c,"myset1",A)
    redis_del(c,"myset2")
    delete A
    A[1]="89"
    redis_sadd(c,"myset2",A)
    redis_del(c,"myset3")
    delete A
    A[1]="9"; A[2]="c16"; A[3]="89"
    redis_sadd(c,"myset3",A)
    delete A
    A[1]="myset1"; A[2]="myset2"; A[3]="myset3"
    redis_sunion(c,A,RE)  # members expeced in array RE: 55, c15, c16, 89, 9

### sunionstore {#sunionstore}
_**Description**_: Adds multiple sets and store the resulting set in a key.

_**Parameters**_   
*number*: connection  
*string*: new key name (a new set), where is stored the result.  
*array*: containing set names  

_**Return value**_   
*number*: the number of elements in the resulting set, or `-1` on error.

{title="Example: Using sunionstore",lang=text,linenos=off}
    redis_del(c,"myset1")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
    redis_sadd(c,"myset1",A)
    redis_del(c,"myset2")
    delete A
    A[1]="89"
    redis_sadd(c,"myset2",A)
    redis_del(c,"myset3")
    delete A
    A[1]="9"; A[2]="c16"; A[3]="89"
    redis_sadd(c,"myset3",A)
    delete A
    A[1]="myset1"; A[2]="myset2"; A[3]="myset3"
    redis_sunionstore(c,"mysetUnion",A)  # members expeced in set mysetUnion: 55, c15, c16, 89, 9

### sdiffstore {#sdiffstore}
_**Description**_: Substracts multiple sets and store the resulting set in a key.

_**Parameters**_   
*number*: connection  
*string*: new key name (a new set), where is stored the result.  
*array*: containing set names  

_**Return value**_   
*number*: the number of elements in the resulting set, or `-1` on error.

{title="Example: Using sdiffstore",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      print ERRNO
      redis_del(c,"myset1")
      A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
      redis_sadd(c,"myset1",A)
      redis_del(c,"myset2")
      delete A
      A[1]="89"
      redis_sadd(c,"myset2",A)
      redis_del(c,"myset3")
      delete A
      A[1]="9"; A[2]="c16"; A[3]="89"
      redis_sadd(c,"myset3",A)
      delete A
      A[1]="myset1"; A[2]="myset2"; A[3]="myset3"
       # the next sdiffstore should returns 2
      redis_sdiffstore(c,"mysetDiff",A)  # members expeced in set mysetDiff: 55,c15
       # for to show the results
      ret=redis_smembers(c,"mysetDiff",MEMB) # 'ret' contains the return of 'smembers' of the resulting set 
       # now, the array MEMB contains the results
      for(i in MEMB) {
        print MEMB[i]
      }
      redis_close(c)
}

### sinterstore {#sinterstore}
_**Description**_: Intersects multiple sets and store the resulting set in a key.

_**Parameters**_   
*number*: connection  
*string*: new key name (a new set), where is stored the result.  
*array*: containing set names  

_**Return value**_   
*number*: the number of elements in the resulting set, or `-1` on error.

{title="Example: Using sinterstore",lang=text,linenos=off}
    redis_del(c,"myset1")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c15"
    redis_sadd(c,"myset1",A)
    redis_del(c,"myset2")
    delete A
    A[1]="89"
    redis_sadd(c,"myset2",A)
    redis_del(c,"myset3")
    delete A
    A[1]="9"; A[2]="c16"; A[3]="89"
    redis_sadd(c,"myset3",A)
    delete A
    A[1]="myset1"; A[2]="myset2"; A[3]="myset3"
    redis_sinterstore(c,"mysetInter",A)  # members expeced in set mysetInter: 89


### sscan {#sscan}
_**Description**_: iterates elements of Sets types. Please read how it works from Redis [sscan](http://redis.io/commands/sscan) command.

_**Parameters**_   
*number*: connection  
*string*: key name  
*number*: the cursor  
*array*:  for to hold the results  
*(optional) string*: for to `match` a given glob-style pattern, similarly to the behavior of the `keys` function that takes a pattern as only argument

_**Return value**_   
`1` on success or `0` on the last iteration (when the returned cursor is equal 0). Returns `-1` on error (by example a WRONGTYPE Operation).

{title="Example: Using sscan",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      num=0
      while(1){
       ret=redis_sscan(c,"myset",num,AR)
       if(ret==-1){
        print ERRNO
        redis_close(c)
        exit
       }
       if(ret==0){
         break
       }  
       n=length(AR)
       for(i=2;i<=n;i++) {
         print AR[i]
       }
       num=AR[1]  # AR[1] contains the cursor
       delete(AR)
      }
      for(i=2;i<=length(AR);i++) {
         print AR[i]
      }
      redis_close(c)
    }

### sadd {#sadd}
_**Description**_: Add one or more members to a set.

_**Parameters**_   
*number*: connection  
*string*: key name  
*string or array*: containing the value, and if it is an array containing the set of values   

_**Return value**_   
`the number of members` added to the set in this operation. Returns `-1` on error (by example a WRONGTYPE Operation).

{title="Example: Using sadd",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      redis_del(c,"myset")
      r=redis_sadd(c,"myset","c15")
      print r
      A[1]="55"; A[2]="c16"; A[3]="89"
      print redis_sadd(c,"myset",A)
      r=redis_smembers(c,"myset",MEMB)
      if(r!=-1) {
         print "Members in set 'myset'"
         for( i in MEMB) {
            print MEMB[i]
         }
      }
      redis_close(c)
    }


{title="Output",lang=text,linenos=off}
    1
    3
    Members in set 'myset'
    89
    c16
    55
    c15

### srem {#srem}
_**Description**_: Remove one or more members from a set.

_**Parameters**_   
*number*: connection  
*string*: key name  
*string or array*: containing the member (a string), and if it is an array containing the set of members (one or more strings)   

_**Return value**_   
*number*: the number of members that were removed from the set. Returns `-1` on error.

{title="Example: Using srem",lang=text,linenos=off}
    redis_del(c,"myset")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c26"; A[5]="12"
    redis_sadd(c,"myset",A)
    r1=redis_srem(c,"myset","89")
    B[1]=55; B[2]="c16"; B[3]="12"
    r2=redis_srem(c,"myset",B)
    print "r1="r1" - r2="r2
    redis_smembers(c,"myset",MEMB)  # member expected in 'myset': c26


### sismember {#sismember}
_**Description**_: Determines if a given value is a member of a set.

_**Parameters**_   
*number*: connection  
*string*: key name, (the set)  
*string*: member

_**Return value**_   
*number*: `1` if the element is a member of the set. `0` if the element is not a member of the set, or if key does not exist.

{title="Example: Using sismember",lang=text,linenos=off}
    redis_del(c,"myset")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c26"; A[5]="12"
    redis_sadd(c,"myset",A)
    redis_sismember(c,"myset","c26") # return value expected: 1
    redis_sismember(c,"myset","66") # return value expected: 0


### smove {#smove}
_**Description**_: Move a member from one set to another.

_**Parameters**_   
*number*: connection  
*string*: key name, the source set  
*string*: key name, the destination set  
*string*: member

_**Return value**_   
`1` if the elemment is moved, `0` if the element is not a member of source and no operation was performed. Returns `-1` on error.

{title="Example: Using smove",lang=text,linenos=off}
    redis_del(c,"myset")
    A[1]="55"; A[2]="c16"; A[3]="89"; A[4]="c26"; A[5]="12"
    redis_sadd(c,"myset",A)
     # execute "smove" and display its return
    print "Member 'c26' from 'myset' to 'newset': "redis_smove(c,"myset","newset","c26")
     # now, the expected return value is 0
    print "Member 'ccc' from 'myset' to 'newset': "redis_smove(c,"myset","newset","ccc")

### scard {#scard}
_**Description**_: Gets the cardinality (number of elements) of the set.

_**Parameters**_   
*number*: connection  
*string*: key name  

_**Return value**_   
`the cardinality` or 0 if key does not exist. `-1` on error.

{title="Example: Using scard",lang=text,linenos=off}
    print "Cardinality of 'myset': "redis_scard(c,"myset")

### smembers {#smembers}
_**Description**_: Gets all the members in a set.

_**Parameters**_   
*number*: connection  
*string*: key name  
*array*: will contain the results, a set of strings.

_**Return value**_   
`1` on success, `-1` on error.

    To see example `sadd function`

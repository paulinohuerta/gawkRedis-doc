# Sorted sets {#sorted-sets}

1. [zadd](#zadd) - Adds one or more members to a sorted set or updates its score if it already exists
1. [zcard](#zcard) - Gets the number of members in a sorted set
1. [zcount](#zcount) - Counts the members in a sorted set with scores between the given values
1. [zincrby](#zincrby) - Increments the score of a member in a sorted set
1. [zinterstore](#zinterstore) - Intersects multiple sorted sets and store the resulting sorted set in a new key
1. [zlexcount](#zlexcount) - Returns the number of elements with a value in a specified range, forcing lexicographical ordering
1. [zrange](#zrange) - Returns a range of members in a sorted set. The elements are sorted from the lowest to the highest score
1. [zrangebylex](#zrangebylex) - Returns all the elements with a value in a specified range, forcing a lexicographical ordering
1. [zrangebyscore](#zrangebyscore) - Returns all the elements with a score between min and max specified
1. [zrangeWithScores](#zrangewithscores) - Return a range of members in a sorted set, by score
1. [zrank](#zrank) - Determines the index of a member in a sorted set
1. [zrem](#zrem) - Removes one or more members from a sorted set
1. [zremrangebylex](#zremrangebylex) - Removes all elements in the sorted set between the lexicographical range specified by min and max
1. [zremrangebyrank](#zremrangebyrank) - Removes all elements in the sorted set with rank into a specified rango
1. [zremrangebyscore](#zremrangebyscore) - Removes all elements in the sorted set with a score into a specified range
1. [zrevrange](#zrevrange) - Returns a specified range of elements in the sorted set. The elements are sorted from highest to lowest score
1. [zrevrangebyscore](#zrevrangebyscore) - Returns all the elements in the sorted set with a score between max and min. 
1. [zrevrangeWhithScores](#zrevrangewithscores) - Executes zrevrange with the option 'withscores', gettings the scores together with the elements
1. [zrevrank](#zrevrank) - Returns the rank of a member in the sorted set, with the scores ordered from high to low
1. [zscan](#zscan) - Iterates elements of Sorted Set types
1. [zscore](#zscore) - Gets the score associated with the given member in a sorted set
1. [zunionstore](#zunionstore) - Adds multiple sorted sets and stores the resulting sorted set in a new key

### zcard {#zcard}
_**Description**_: Gets the number of members in a sorted set.

_**Parameters**_    
*number*: connection  
*string*: key name  

_**Return value**_   
*number*: `the cardinality` or number de elements, `0` if key does not exist. `-1` on error.

{title="Example: Using zcard",lang=text,linenos=off}
    print "Cardinality of 'zmyset':",
           redis_zcard(c,"zmyset")

### zrevrank {#zrevrank}
_**Description**_: Returns the rank of a member in the sorted set, with the scores ordered from high to low. The rank (or index) is 0-based, which means that the member with the highest score has rank 0.

_**Parameters**_    
*number*: connection  
*string*: key name   
*string*: member name   

_**Return value**_   
*number*: the rank of member, if member exists in the sorted set. Returns `null string` if member does not exist in the sorted set or key does not exist. `-1` on error.

{title="Example: Using zrevrank",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      A[1]="1"; A[2]="one"; A[3]="2"; A[4]="two"
      A[5]="3"; A[6]="three"; A[7]="4"; A[8]="four"
      redis_zadd(c,"myzset",A)
      redis_zrevrank(c,"myzset","four") # returns 0
      redis_zrevrank(c,"myzset","seven")
       # and returns null string
      redis_zrevrank(c,"myzset","two") # returns 2
      redis_close(c)
    }

### zcount {#zcount}
_**Description**_: Count the members in a sorted set with a score between min and max (two values given as arguments).

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: min value
*number*: max value

_**Return value**_   
*number*: the number of elements in the specified score range, `0` if key does not exist. `-1` on error (by example a WRONGTYPE Operation).

{title="Example: Using zcount",lang=text,linenos=off}
    redis_del(c,"zmyset")
    r1=redis_zadd(c,"zmyset",1,"one")
    r2=redis_zadd(c,"zmyset",1,"uno")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three"
    r3=redis_zadd(c,"zmyset",AR)
    print r1, r2, r3
    print "Zcount with score between 1 and 2:",
          redis_zcount(c,"zmyset",1,2) # returns 3

### zinterstore {#zinterstore}
_**Description**_: Intersects multiple sorted sets and store the resulting sorted set in a new key.
To see [Redis site](http://redis.io/commands/zinterstore) for to know how use additionals parameters "weights" and "aggregate"

_**Parameters**_    
*number*: connection  
*string*: new key name (a new sorted set), where is stored the result.  
*array*: containing the sorted sets names  
`and optionally...`
*array*: containing the weights 
*string*: containing "agregate sum|min|max"  
 
_**Return value**_   
*number*: the number of elements in the resulting sorted set at destination, or `-1` on error

{title="Example: Using zinterstore",lang=text,linenos=off}
    redis_del(c,"zmyset1")
    A[1]="1"; A[2]="one"; A[3]="3"; A[4]="three"
    A[5]="5"; A[6]="five"
    redis_zadd(c,"zmyset1",A)
    redis_del(c,"zmyset2")
    delete A
    A[1]="3"; A[2]="three"; A[3]="4"; A[4]="four"
    redis_zadd(c,"zmyset2",A)
    redis_del(c,"zmyset3")
    delete A
    A[1]="3"; A[2]="three"; A[3]="4"; A[4]="four"
    A[5]="5"; A[6]="five"
    redis_zadd(c,"zmyset3",A)
    delete A
    A[1]="zmyset1"; A[2]="zmyset2"; A[3]="zmyset3"
    redis_zinterstore(c,"zmysetInter",A)
    # members expected in sorte set zmysetInter: 'three'
    # with score 9
    W[1]=2; W[2]=3; W[3]=4
    redis_zinterstore(c,"zmysetInterWeights",A,W,"aggregate sum")
    # 'three' with score 27
    redis_zinterstore(c, "zmysetInterWeights", A,W, "aggregate min")
    # 'three' with score 6

### zunionstore {#zunionstore}
_**Description**_: Adds multiple sorted sets and store the resulting sorted set in a new key.
To see [Redis site](http://redis.io/commands/zunionstore) for to know how use additionals parameters "weights" and "aggregate"

_**Parameters**_    
*number*: connection  
*string*: new key name (a new sorted set), where is stored the result.  
*array*: containing the sorted sets names  
and optionally:  
*array*: containing the weights 
*string*: containing "agregate sum|min|max"  
 
_**Return value**_   
*number*: the number of elements in the resulting sorted set at destination, or `-1` on error

{title="Example: Using zunionstore",lang=text,linenos=off}
    redis_del(c,"zmyset1")
    A[1]="1"; A[2]="one"; A[3]="3"; A[4]="three"
    A[5]="5"; A[6]="five"
    redis_zadd(c,"zmyset1",A)
    redis_del(c,"zmyset2")
    delete A
    A[1]="3"; A[2]="three"; A[3]="4"; A[4]="four"
    redis_zadd(c,"zmyset2",A)
    redis_del(c,"zmyset3")
    delete A
    A[1]="3"; A[2]="three"; A[3]="4"; A[4]="four"
    A[5]="5"; A[6]="five"
    redis_zadd(c,"zmyset3",A)
    delete A
    A[1]="zmyset1"; A[2]="zmyset2"; A[3]="zmyset3"
    W[1]=2; W[2]=3; W[3]=4
    redis_zunionstore(c,"zmysetUW",A,W,"aggregate sum")
     # one,2  three,27  four,28  five,30 
    redis_zunionstore(c,"zmysetUW",A,W,"aggregate min")
     # one,2  three,6  four,12  five,10


### zrange {#zrange}
_**Description**_: Returns a range of members in a sorted set. The members are considered to be ordered from the lowest to the highest score.

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*:  array name for the results  
*number*: start of range  
*number*: stop of range  

_**Return value**_   
`1` on success, `0` if the result is empty, `-1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrange",lang=text,linenos=off}
    redis_del(c,"zmyset")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three";
    AR[5]="1"; AR[6]="one"; AR[7]="1"; AR[8]="uno"
    redis_zadd(c,"zmyset",AR)
    redis_zrange(c,"zmyset",RET,6,-1) 
     # returns 0, and array RET is empty
    redis_zrange(c,"zmyset",RET,1,2) 
     # returns 1, and array RET contains members
     #
     # shows the results
    for( i in RET ) {
      print RET[i]
    }

### zrevrange {#zrevrange}
_**Description**_: Returns the specified range of elements in the sorted set. The elements are considered to be ordered from the highest to the lowest score

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*: array name for the results  
*number*: start of range  
*number*: stop of range  

_**Return value**_   
`1` on success, `0` if the result is empty, or the key not exists. `-1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrevrange",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","1","t7")
      redis_zadd(c,"myzset","2","t0")
      redis_zadd(c,"myzset","5","t1")
      redis_zadd(c,"myzset","4","t9")
       #  zrevrange(c,"myzset",RES,6,-1)  # returns 0
      print redis_zrevrange(c,"myzset",RES,0,-1)# returns 1
      for (i in RES) {
         print i": "RES[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1
    1: t1
    2: t9
    3: t0
    4: t7

### zrevrangeWithScores {#zrevrangewithscores}
_**Description**_: Returns the specified range of elements in the sorted set. The elements are considered to be ordered from the highest to the lowest score. Returns  the scores of the elements together with the elements. 

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*: array name for the results. It will contain value1,score1,..., valueN,scoreN instead value1,...valueN  
*number*: start of range  
*number*: stop of range  

_**Return value**_   
`1` on success, `0` if the result is empty, or the key not exists. `-1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrevrangeWithScores",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","1","t7")
      redis_zadd(c,"myzset","2","t0")
      redis_zadd(c,"myzset","5","t1")
      redis_zadd(c,"myzset","4","t9")
      redis_zrevrangeWithScores(c,"myzset",RES,1,-1)  # returns 1
      for (i in RES) {
         print i": "RES[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1: t9
    2: 4
    3: t0
    4: 2
    5: t7
    6: 1


### zlexcount
_**Description**_: When all the elements in a sorted set are inserted with the same score, returns the number of elements with a value between min and max specified, forcing lexicographical ordering.
To see [the Redis command](http://redis.io/commands/zlexcount) to know how to specify intervals and others details.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: min  
*string*: max  

_**Return value**_   
*number*: the number of elements in the specified score range. `-1` on error

{title="Example: Using zlexcount",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"zset")
      A[1]="0"; A[2]="a"; A[3]="0"; A[4]="b"; A[5]="0"
      A[6]="c"; A[7]="0"; A[8]="d"; A[9]="0"; A[10]="e"
      A[11]="0"; A[12]="f"; A[13]="0"; A[14]="g"
      redis_zadd(c,"zset",A)
      redis_zlexcount(c,"zset","-","+")  # return 7
      redis_zlexcount(c,"zset","[b","(d")  # returns 2
      redis_close(c)
    }

### zremrangebylex {#zremrangebylex}
_**Description**_: When all the elements in a sorted set are inserted with the same score, removes all elements in the sorted set between the lexicographical range specified by min and max
To see [the Redis command](http://redis.io/commands/zremrangebylex) to know how to specify intervals and others details.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: min  
*string*: max  

_**Return value**_   
*number*: the number of elements removed. `-1` on error

{title="Example: Using zremrangebylex",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","2","one")
      redis_zadd(c,"myzset","2","two")
      redis_zadd(c,"myzset","2","three")
      redis_zremrangebylex(c,"myzset","[g","(tkz") # returns 2
      redis_zrange(c,"myzset",RES,0,-1) # returns 1
      for (i in RES) {
        print i": "RES[i]
      }
    }

### zremrangebyscore {#zremrangebyscore}
_**Description**_: Removes all elements in the sorted set with a score into a specified range with a min and a maxm (inclusive).
To see [the Redis command](http://redis.io/commands/zremrangebyscore) to know how to specify intervals and others details.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: min  
*string*: max  

_**Return value**_   
*number*: the number of elements removed. `-1` on error

{title="Example: Using zremrangebyscore",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","1","t7")
      redis_zadd(c,"myzset","2","t0")
      redis_zadd(c,"myzset","5","t1")
      redis_zadd(c,"myzset","4","t9")
       # redis_zremrangebyscore(c,"myzset","-inf","(5")
       # returns 3
       # redis_zremrangebyscore(c,"myzset",1,3)# returns 2
      redis_zremrangebyscore(c,"myzset","(2","4")#returns 1
      redis_zrangeWithScores(c,"myzset",RES,0,-1)
       # returns 1, and the results in array RES
      for (i in RES) {
         print i": "RES[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1: t7
    2: 1
    3: t0
    4: 2
    5: t1
    6: 5

### zremrangebyrank {#zremrangebyrank}
_**Description**_: Removes all elements in the sorted set with rank between start and stop. Both start and stop are 0 -based indexes with 0 being the element with the lowest score.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: min  
*string*: max  

_**Return value**_   
*number*: the number of elements removed. `-1` on error

{title="Example: Using zremrangebyrank",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","1","t7")
      redis_zadd(c,"myzset","2","t0")
      redis_zadd(c,"myzset","5","t1")
      redis_zadd(c,"myzset","4","t9")
      redis_zremrangebyrank(c,"myzset",0,1) # returns 2
      redis_zrangeWithScores(c,"myzset",RES,0,-1)  # returns 1
      for (i in RES) {
        print i": "RES[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1: t9
    2: 4
    3: t1
    4: 5

### zrangebylex {#zrangebylex}
_**Description**_: When all the elements in a sorted set are inserted with the same score, returns all the elements with a value between min and max specified, forcing a lexicographical ordering.
To see [the Redis command](http://redis.io/commands/zrangebylex) to know how to specify intervals and others details.

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*: for the results, will be a list of elements with value in the specified range.
*string*: min  
*string*: max  

_**Return value**_   
`1` when obtains results,`0` when list empty (no elements in the score range) or the key name no exists, `1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrangebylex",lang=text,linenos=off}
    c=redis_connect()
    redis_del(c,"zset")
    A[1]="0"; A[2]="a"; A[3]="0"; A[4]="b"; A[5]="0"
    A[6]="c"; A[7]="0"; A[8]="d"; A[9]="0"; A[10]="e"
    A[11]="0"; A[12]="f"; A[13]="0"; A[14]="g"
    redis_zadd(c,"zset",A)  # returns 7
    redis_zrangebylex(c,"zset",AR,"[aaa","(g") # returns 1
     # AR contains b,c,d,e,f
     # to show the result contained in array AR
    for(i in AR){
      print i": "AR[i]
    }
     # the next return is 0
    redis_zrangebylex(c,"zset",AR,"[pau","(ra")
     # the array has not content

### zrangebyscore {#zrangebyscore}
_**Description**_: Returns all the elements with a score between min and max specified. The elements are considered to be ordered from low to high scores.
To see [the Redis command](http://redis.io/commands/zrangebyscore) to know how to specify intervals and others details.

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*: for the results, will be a list of elements in the specified score range.
*string*: min  
*string*: max  

_**Return value**_   
`1` when obtains results,`0` when list empty (no elements in the score range) or the key name no exists, `1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrangebyscore",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","1","one")
      redis_zadd(c,"myzset","2","two")
      redis_zadd(c,"myzset","3","three")
      redis_zrangebyscore(c,"myzset",RES,"-inf","+inf")
        # returns 1
      for (i in RES) {
         print i": "RES[i]
      }
      delete RES
      redis_zrangebyscore(c,"myzset",RES,1,2) # returns 1
      for (i in RES) {
         print i": "RES[i]
      }
      redis_zrangebyscore(c,"myzset",RES,"(1","(2")
      # returns 0
      redis_close(c)
    }

### zrevrangebyscore {#zrevrangebyscore}
_**Description**_: Returns all the elements in the sorted set with a score between max and min (including elements with score equal to max or min). The elements are sorted from highest to lowest score.
To see [the Redis command](http://redis.io/commands/zrevrangebyscore) to know how to specify intervals and others details.

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*: for the results, will be a list of elements in the specified score range.
*string*: max  
*string*: min  

_**Return value**_   
`1` when obtains results,`0` when list empty (no elements in the score range) or the key name no exists, `1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrevrangebyscore",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"myzset")
      redis_zadd(c,"myzset","1","one")
      redis_zadd(c,"myzset","2","two")
      redis_zadd(c,"myzset","3","three")
      redis_zrevrangebyscore(c,"myzset",RES,"+inf","-inf")
       # returns 1
      for (i in RES) {
         print i": "RES[i]
      }
      delete RES
      redis_zrevrangebyscore(c,"myzset",RES,"2","1")
       # returns 1
      print
      for (i in RES) {
         print i": "RES[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1: three
    2: two
    3: one
    
    1: two
    2: one

### zrangeWithScores {#zrangewithscores}
_**Description**_: Returns the scores of the elements together with the elements in a range, in a sorted set.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: array name for the results. It will contain value1,score1,..., valueN,scoreN instead value1,...valueN  
*number*: start of range  
*number*: stop of range  

_**Return value**_   
*number*: `1` on success, `0` if the result is empty, `-1` on error (by example a WRONGTYPE Operation)

{title="Example: Using zrangeWithScores",lang=text,linenos=off}
    redis_del(c,"zmyset")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three";
    AR[5]="1"; AR[6]="one"; AR[7]="1"; AR[8]="uno"
    redis_zadd(c,"zmyset",AR)
    redis_zrange(c,"zmyset",RET,0,-1) #  gets only elements
     # use RET ... and then remove 
    delete RET
    redis_zrangeWithScores(c,"zmyset",RET,0,-1)
     # gets all elements with their respectives scores
     #
     # shows the results
    for( i in RET ) {
      print RET[i]
    }

### zrem {#zrem}
_**Description**_: Removes one or more members from a sorted set.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string or array*: the member (a string) or the set of members that containing the array 

_**Return value**_   
*number*: The number of members removed from the sorted set, `-1`on error.

{title="Example: Using zrem",lang=text,linenos=off}
    redis_del(c,"zmyset")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three"
    AR[5]="1"; AR[6]="one"
    redis_zadd(c,"zmyset",AR)
    redis_zrem(c,"zmyset","three") # returns 1
    R[1]="uno"; R[2]="two"; R[3]="five"
    redis_zrem(c,"zmyset",R) # returns 2

### zrank {#zrank}
_**Description**_: Determines the index or rank of a member in a sorted set.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: the member

_**Return value**_   
`the rank of member`, if the member exists in the key. `string null`, if the member does not exist in the key or the key does not exist, `-1`on error.

{title="Example: Using zrank",lang=text,linenos=off}
    redis_del(c,"zmyset")
    redis_zadd(c,"zmyset",1,"uno")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three"
    AR[5]="1"; AR[6]="one"
    redis_zadd(c,"zmyset",AR)
    redis_zrank(c,"zmyset","three") # returns 3
    redis_zrank(c,"zmyset","one") # returns 0

### zcore {#zscore}
_**Description**_: Gets the score associated with the given member in a sorted set.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: the member

_**Return value**_   
`the score of member represented as string`, if the member exists in the key. `string null`, if the member does not exist in the key or the key does not exist. `-1`on error.

{title="Example: Using zscore",lang=text,linenos=off}
    redis_del(c,"zmyset")
    redis_zadd(c,"zmyset",1,"uno")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three"
    AR[5]="1"; AR[6]="one"
    redis_zadd(c,"zmyset",AR)
    redis_zscore(c,"zmyset","three") # returns 3
    redis_zscore(c,"zmyset","one") # returns 1

### zincrby {#zincrby}
_**Description**_: Increments the score of a member in a sorted set.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: the increment  
*string*: the member

_**Return value**_   
*number*: the new score of member.

{title="Example: Using zincrby",lang=text,linenos=off}
    redis_del(c,"zmyset")
    redis_zadd(c,"zmyset",1,"uno")
    AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three"
    AR[5]="1"; A[6]="one"
    redis_zadd(c,"zmyset",AR)
    # redis_zincrby increments '3' the score of the
    # member 'one' of key 'zmyset'
    redis_zincrby(c,"zmyset",3,"one") # returns 4

### zadd {#zadd}
_**Description**_: Adds one or more members to a sorted set or updates its score if it already exists.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: score
*string or array*: containing the member, and if it is an array containing the set of score and members   

_**Return value**_   
`the number of elements` added to the sorted set, not including elements already existing. Returns `-1` on error (by example a WRONGTYPE Operation).

{title="Example: Using zadd",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      redis_del(c,"zmyset")
      r1=redis_zadd(c,"zmyset",1,"one")
      r2=redis_zadd(c,"zmyset",1,"uno")
      AR[1]="2"; AR[2]="two"; AR[3]="3"; AR[4]="three"
      r3=redis_zadd(c,"zmyset",AR)
      print r1, r2, r3
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1 1 2

### zscan {#zscan}
_**Description**_: iterates elements of Sets types. Please read how it works from Redis [zscan](http://redis.io/commands/zscan) command.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: the cursor  
*array*:  for to hold the results  
*string (optional)*: for to `match` a given glob-style pattern, similarly to the behavior of the `keys` function that takes a pattern as only argument

_**Return value**_   
`1` on success or `0` on the last iteration (when the returned cursor is equal 0). Returns `-1` on error (by example a WRONGTYPE Operation).

{title="Example: Using zscan",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      num=0
      while(1){
       ret=redis_zscan(c,"myzset1",num,AR)
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


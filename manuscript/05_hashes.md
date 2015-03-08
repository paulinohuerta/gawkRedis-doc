# Hashes Functions {#hashes}

1. [hdel](#hdel) - Deletes one or more hash fields
1. [hexists](#hexists) - Determines if a hash field exists
1. [hget](#hget) - Gets the value of a hash field
1. [hgetAll](#hgetall) - Gets all the fields and values in a hash
1. [hincrby](#hincrby) - Increments the integer value of a hash field by the given number
1. [hincrbyfloat](#hincrbyfloat) - Increments the float value of a hash field by the given amount
1. [hkeys](#hkeys) - Gets all the fields in a hash
1. [hlen](#hlen) - Gets the number of fields in a hash
1. [hmget](#hmget) - Gets the values of all the given hash fields
1. [hmset](#hmset) - Sets multiple hash fields to multiple values
1. [hset](#hset) - Sets the string value of a hash field
1. [hsetnx](#hsetnx) - Sets the value of a hash field, only if the field does not exist
1. [hscan](#hscan) - Iterates elements of Hash types
1. [hvals](#hvals) - Gets all the values in a hash

### hset
_**Description**_: Adds a value to the hash stored at key. If this value is already in the hash, `FALSE` is returned.  

_**Parameters**_    
*number*: connection  
*string*: key name.  
*string*: hash Key   
*string*: value  

_**Return value**_    
`1` if value didn't exist and was added successfully, `0` if the value was already present and was replaced, `-1` if there was an error.

{title="Example: Using hset",lang=text,linenos=off} 
    @load "redis"
    BEGIN{
     c=redis_connect()
     redis_del(c,"thehash")
     redis_hset(c,"thehash","key1","hello") # returns 1
     redis_hget(c,"thehash", "key1") # returns "hello"
     redis_hset(c,"thehash", "key1", "plop") # returns 0, value was replaced
     redis_hget(c,"thehash", "key1") # returns "plop"
     redis_close(c)
    }

### hsetnx
_**Description**_: Adds a value to the hash stored at key only if this field isn't already in the hash.

_**Return value**_    
`1` if the field was set, `0` if it was already present.

{title="Example: Using hsetnx",lang=text,linenos=off} 
    redis_del(c,"thehash")
    redis_hsetnx(c,"thehash","key1","hello") # returns 1
    redis_hget(c,"thehash", "key1") # returns "hello"
    redis_hsetnx(c,"thehash", "key1", "plop") # returns 0. No change, value wasn't replaced
    redis_hget(c,"thehash", "key1") # returns "hello"

### hget
_**Description**_: Gets a value associated with a field from the hash stored it key.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: hash field  

_**Return value**_    
*string*: the value associated with field, or `string null` when field is not present in the hash or the key does not exist.


### hlen {#hlen}
_**Description**_: Returns the length of a hash, in number of items

_**Parameters**_    
*number*: connection  
*string*: key name  

_**Return value**_    
*number*: the number of fields in the hash, or `0` when key does not exist. `-1` on error (by example if key exist and isn't a hash).

{title="Example: Using hlen",lang=text,linenos=off} 
    redis_hsetnx(c,"thehash","key1","hello1") # returns 1
    redis_hsetnx(c,"thehash","key2","hello2") # returns 1
    redis_hsetnx(c,"thehash","key3","hello3") # returns 1
    redis_hlen(c,"thehash")  # returns 3

### hdel
_**Description**_: Removes the specified fields from the hash stored at key.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string or array*: field name, or array name containing the field names 

_**Return value**_    
*number*: the number of fields that were removed from the hash, not including specified but non existing fields.


### hkeys
_**Description**_: Obtains the keys in a hash.

_**Parameters**_    
*number*: connection  
*Key*: key name  
*array*: containing field names results

_**Return value**_    
`1` on success, `0` if the hash is empty or no exists

{title="Example: Using hkeys",lang=text,linenos=off} 
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_hkeys(c,"thehash",A) # returns 1
      for(i in A){
        print i": "A[i]
      }
      redis_close(c)
    }

The order is random and corresponds to redis' own internal representation of the structure.

### hvals
_**Description**_: Obtains the values in a hash.

_**Parameters**_    
*number*: connection  
*Key*: key name  
*array*: contains the result with values

_**Return value**_    
`1` on success, `0` if the hash is empty or no exists

{title="Example: Using hvals",lang=text,linenos=off} 
    @load "redis"
    BEGIN{
     c=redis_connect()
     redis_hvals(c,"thehash",A) # returns 1
     for(i in A){
       print i": "A[i]
     }
     redis_close(c)
    }

The order is random and corresponds to redis' own internal representation of the structure.

### hgetall
_**Description**_: Returns the whole hash.

_**Parameters**_    
*number*: connection  
*Key*: key name  
*array*: for the result, contains the entire sequence of field/value

_**Return value**_    
`1` on success, `0` if the hash is empty or no exists

{title="Example: Using hgetall",lang=text,linenos=off} 
    @load "redis"
    BEGIN{
    c=redis_connect()
    redis_hgetall(c,"thehash",A) # returns 1
    n=length(A)
    for(i=1;i<=n;i+=2){
      print i": "A[i]" ---> "A[i+1]
    }
    redis_close(c)
    }

The order is random and corresponds to redis' own internal representation of the structure.

### hscan {#hscan}
_**Description**_: iterates elements of Hash types. Please read how it works from Redis [hscan](http://redis.io/commands/hscan) command.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: the cursor  
*array*:  for to hold the results  
*string (optional)*: for to `match` a given glob-style pattern, similarly to the behavior of the `keys` function that takes a pattern as only argument

_**Return value**_    
`1` on success,  or `0` on the last iteration (when the returned cursor is equal 0). Returns `-1` on error (by example a WRONGTYPE Operation).

{title="Example: Using hscan",lang=text,linenos=off} 
    @load "redis"
    BEGIN{
      c=redis_connect()
      num=0
      while(1){
        ret=redis_hscan(c,"myhash",num,AR)
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

### hexists
_**Description**_: Verify if the specified member exists in a hash.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: field or member  

_**Return value**_    
`1` If the member exists in the hash, otherwise return `0`.

{title="Example: Using hexists",lang=text,linenos=off} 
    @load "redis"
    BEGIN {
      c=redis_connect()
      if(redis_hexists(c,"hashb","cl1")==1) {
        print "Key cl1 exists in the hash hashb"
      }
      if(redis_hexists(c,"hashb","cl1")==0) {
        print "Key cl1 does not exist in the hash hashb"
      }
      if(redis_hexists(c,"hashb","cl1")==-1) {
        print ERRNO
      }
      redis_close(c)
    }

### hincrby
_**Description**_: Increments the value of a member from a hash by a given amount.

_**Parameters**_    
##### *Parameters*
*number*: connection  
*string*: key name  
*string*: member or field    
*number*: (integer) value that will be added to the member's value  

_**Return value**_    
##### *Return value*
*number*: the new value

{title="Example: Using hincrby",lang=text,linenos=off} 
    print redis_hset(c,"hashb","field", 5)     # returns 1
    print redis_hincrby(c,"hashb","field", 1)  # returns 6
    print redis_hincrby(c,"hashb","field", -1) # returns 5
    print redis_hincrby(c,"hashb","field", -10)# returns -5


### hincrbyfloat
_**Description**_: Increments the value of a hash member by the provided float value

_**Parameters**_    
*number*:connection  
*string*: key name   
*string*: field name   
*number*: (float) value that will be added to the member's value  

_**Return value**_    
*number*: the new value

{title="Example: Using hincrbyfloat",lang=text,linenos=off} 
    redis_del(c,"h");
    redis_hincrbyfloat(c,"h","x",1.5)
      # returns 1.5: field x = 1.5 now
    redis_hincrbyfloat(c,"h","x", 1.5)
      # returns 3.0: field x = 3.0 now
    redis_hincrbyfloat(c,"h","x",-3.0)
      # returns 0.0: field x = 0.0 now

### hmset
_**Description**_: Fills in a whole hash. Overwriting any existing fields in the hash. If key does not exist, a new key holding a hash is created.

_**Parameters**_    
*number* connection  
*string*: key name  
*array*: contains field names and their respective values

_**Return value**_    
`1` on success, `-1` on error

{title="Example: Using hmset",lang=text,linenos=off} 
    c=redis_connect()
    AR[1]="a0"
    AR[2]="value of a0"
    AR[3]="a1"
    AR[4]="value of a1"
    ret=redis_hmset(c,"hash1",AR1)

### hmget
_**Description**_: Retrieve the values associated to the specified fields in the hash.

_**Parameters**_    
*number*: connection  
*string*: key name  
*array or string*: an array contains field names, or only one string that containing the name of field   
*array*: contains results, a sequence of values associated with the given fields, in the same order as they are requested. For every field that does not exist in the hash, a null string (empty string) is associated.

_**Return value**_    
`1` on success, `-1` on error

{title="Example: Using hmget",lang=text,linenos=off} 
    load "redis"
    BEGIN{
     c=redis_connect()
     J[1]="c2"
     J[2]="k3"
     J[3]="cl1"
     J[4]="c1"
     J[5]="c6"
     ret=redis_hmget(c,"thash",J,T)
     if(ret==-1) {
       print ERRNO
     }
     print "hmget: Results and requests"
     for (i in T) {
       print i": ",T[i], " ........ ",J[i]
     }
     ret=redis_hgetall(c,"thash",AR)
     print "hgetall from the hash thash"
     for (i in AR) {
       print i": "AR[i]
     }
     # other use allowed for hmget
     ret=redis_hmget(c,"thash","cl1",OTH)
     print "is cl1 a field?"
     for(i in OTH){
       print i": "OTH[i]
     }
     redis_close(c);
    }

{title="Output",lang=text,linenos=off} 
    hmget: Results and requests
    1:    ........  c2
    2:  vk3  ........  k3
    3:  vcl1  ........  cl1
    4:    ........  c1
    5:    ........  c6
    hgetall from the hash thash
    1: k1
    2: vk1
    3: k3
    4: vk3
    5: cl1
    6: vcl1
    7: cl2
    8: vcl2
    is cl1 a field?
    1: vcl1


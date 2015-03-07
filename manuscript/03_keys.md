# Keys

1. [del](#del) - Delete a key
* [dump](#dump) - Return a serialized version of the value stored at the specified key.
1. [exists](#exists) - Determine if a key exists
1. [expire, pexpire](#expire-pexpire) - Set a key's time to live in seconds
1. [keys](#keys) - Find all keys matching the given pattern
1. [move](#move) - Move a key to another database
1. [persist](#persist) - Remove the expiration from a key
1. [randomkey](#randomkey) - Return a random key from the keyspace
1. [rename](#rename) - Rename a key
1. [renamenx](#renamenx) - Rename a key, only if the new key does not exist
1. [sort](#sort) - Sort the elements in a list, set or sorted set
1. [sortLimit](#sortlimit) - Sort the elements in a list, set or sorted set, using the LIMIT modifier
1. [sortLimitStore](#sortlimitstore) - Sort the elements in a list, set or sorted set, using the LIMIT and STORE modifiers
1. [sortStore](#sortstore) - Sort the elements in a list, set or sorted set, using the STORE modifier
1. [scan](#scan) - iterates the set of keys in the currently selected Redis db
1. [type](#type) - Determine the type stored at key
1. [ttl, pttl](#ttl-pttl) - Get the time to live for a key
1. [restore](#restore) - Create a key using the provided serialized value, previously obtained with [dump](#dump).

-----

### del
_**Description**_: Remove specified keys.

##### *Parameters*
*string or array of string*: `key name` or `array name` containing the names of the keys

##### *Return value*
*number*: Number of keys deleted.  

{title="Example: Using del",lang=text,linenos=off}
    redis_set(c,"keyX","valX")
    redis_set(c,"keyY","valY")
    redis_set(c,"keyZ","valZ")
    redis_set(c,"keyU","valU")
    AR[1]="keyY"
    AR[2]="keyZ"
    AR[3]="keyU"
    redis_del(c,"keyX") # return 1 
    redis_del(c,AR) # return 3

### exists
_**Description**_: Verify if the specified key exists.

##### *Parameters*
*number*: connection  
*string*: key name

##### *Return value*
`1` If the key exists, `0` if the key no exists.

##### *Example*
{:lang="awk"}
    redis_set(c,"key","value");
    redis_exists(c,"key"); # return 1
    redis_exists(c,"NonExistingKey") # return 0

### randomKey
_**Description**_: Returns a random key.

##### *Parameters*
*number*: connection  

##### *Return value*
*string*: a random key from the currently selected database

##### *Example*
{:lang="awk"}
    print redis_randomkey(c)

### move
_**Description**_: Moves a key to a different database. The key will move only if not exists in destination database.

##### *Parameters*
*number*: connection  
*string*: key, the key to move  
*number*: dbindex, the database number to move the key to  

##### *Return value*
`1` if key was moved, `0` if key was not moved.

##### *Example*
{:lang="awk"}
    redis_select(c,0)	# switch to DB 0
    redis_set(c,"x","42") # write 42 to x
    redis_move(c,"x", 1) # move to DB 1
    redis_select(c,1)	# switch to DB 1
    redis_get(c,"x");	# will return 42

### rename
_**Description**_: Renames a key. If newkey already exists it is overwritten.

##### *Parameters*
*number*: connection  
*string*: srckey, the key to rename.  
*string*: dstkey, the new name for the key.

##### *Return value*
`1` in case of success, `-1` in case of error.

##### *Example*
{:lang="awk"}
    redis_set(c,"x", "valx");
    redis_rename(c,"x","y");
    redis_get(c,"y")  # return "valx"
    redis_get(c,"x")  # return null string, because x no longer exists

### renamenx
_**Description**_: Same as rename, but will not replace a key if the destination already exists. This is the same behaviour as set and option nx.

##### *Return value*
`1` in case of success, `0` in case not success.

### expire, pexpire
_**Description**_: Sets an expiration date (a timeout) on an item. pexpire requires a TTL in milliseconds.

##### *Parameters*
*number*: connection  
*string*: key name. The key that will disappear.  
*number*: ttl. The key's remaining Time To Live, in seconds.

##### *Return value*
`1` in case of success, `0` if key does not exist or the timeout could not be set
##### *Example*
{:lang="awk"}
    ret=redis_set(c,"x", "42")  # ret value 1; x value "42"
    redis_expire(c,"x", 3)      # x will disappear in 3 seconds.
    system("sleep 5")     # wait 5 seconds
    redis_get(c,"x")  # will return null string, as x has expired.

### keys
_**Description**_: Returns the keys that match a certain pattern. Check supported [glob-style patterns](http://redis.io/commands/keys)

##### *Parameters*
*number*: connection  
*string*: pattern  
*array of strings*: the results, the keys that match a certain pattern.

##### *Return value*
`1` in case of success, `-1` on error

##### *Example*
{:lang="awk"}
    redis_keys(c,"*",AR)    # all keys will match this.
    # show AR contains
    delete AR
    redis_keys(c,"user*",AR)  # for matching all keys begining with "user"
    for(i in AR) {
      print i": "AR[i]
    }

### type
_**Description**_: Returns the type of data pointed by a given key.

##### *Parameters*
*number*: connection  
*string*: key name  

##### *Return value*
*string*: the type of the data (string, list, set, zset and hash) or `none` when the key does not exist.

##### *Example*
{:lang="awk"}
    redis_set(c,"keyZ","valZ")
    ret=redis_type(c,"keyZ") # ret contains "string"
    # showing the "type" all keys of DB 4
    redis_select(c,4)
    redis_keys(c,"*",KEYS) 
    for(i in KEYS){
      print i": "KEYS[i]" ---> "redis_type(c,KEYS[i])
    } 

### sort
-----
_**Description**_: Sort the elements in a list, set or sorted set.

##### *Parameters*
*number*: connection  
*string*: key name  
*array*: the array with the result  
*optional string*: options "desc|asc alpha"

##### *Return value*
`1` or `-1`on error 

##### *Example*
{:lang="awk"}
    c=redis_connect()
    redis_del(c,"thelist1");
    print redis_type(c,"thelist1") # none
    redis_lpush(c,"thelist1","bed")
    redis_lpush(c,"thelist1","pet")
    redis_lpush(c,"thelist1","key")
    redis_lpush(c,"thelist1","art")
    redis_lrange(c,"thelist1",AR,0, -1)
    for(i in AR){
      print i") "AR[i]
    }
    delete AR
    # sort desc "thelist1"
    ret=redis_sort(c,"thelist1",AR,"alpha desc")
    print "-----"
    for(i in AR){
      print i") "AR[i]
    }
    print "-----"


### sortLimit
_**Description**_: Sort the elements in a list, set or sorted set, using the LIMIT modifier with the sense of limit the number of returned elements.

##### *Parameters*
*number*: connection  
*string*: key name  
*array*: the array with the result 
*number*: offset
*number*: count
*optional string*: options "desc|asc alpha"

##### *Return value*
`1` or `-1`on error 

##### *Example*
{:lang="awk"}
    #  will return 5 elements of the sorted version of list2, starting at element 0
    c=redis_connect()
    ret=redis_sortLimit(c,"list2",AR,0,5) # assume "list2" with numerical content
     # or using a sixth argument
     # ret=redis_sortLimit(c,"list2",AR,0,5,"desc") for Alphanumeric content should use "alpha"
    if(ret==-1) {
      print ERRNO
    }
    for(i in AR){
      print i") "AR[i]
    }
    redis_close(c)


### sortLimitStore
_**Description**_: Sort the elements in a list, set or sorted set, using the LIMIT and STORE modifiers with the sense of limit the number of returned elements and ensure that the result is stored as in a new key instead of be returned.


##### *Parameters*
*number*: connection  
*string*: key name  
*string*: the name of the new key 
*number*: offset
*number*: count
*otional string*: options "desc|asc alpha"

##### *Return value*
`1` or `-1`on error 

##### *Example*
{:lang="awk"}
    #  will store 5 elements, of the sorted version of list2,
    #  in the list "listb"
    c=redis_connect()
    ret=redis_sortLimitStore(c,"list2","listb",0,5) # assume "list2" with numerical content
    # or using a sixth argument
    # ret=redis_sortLimitStore(c,"list2","listb",0,5,"desc")

### sortStore
_**Description**_: Sort the elements in a list, set or sorted set, using the STORE modifier for that the result to be stored in a new key

##### *Parameters*
*number*: connection  
*string*: key name  
*string*: the name of the new key 
*optional string*: options "desc|asc alpha"

##### *Return value*
`1` or `-1`on error 

##### *Example*
{:lang="awk"}
    c=redis_connect()
    redis_del(c,"list2")
    redis_lpush(c,"list2","John")
    redis_lpush(c,"list2","Sylvia")
    redis_lpush(c,"list2","Tom")
    redis_lpush(c,"list2","Brenda")
    redis_lpush(c,"list2","Charles")
    redis_lpush(c,"list2","Liza")
    ret=redis_sortStore(c,"list2","listb")
    # or using a fourth argument
    # ret=redis_sortStore(c,"list2","listb","desc alpha")


### scan
_**Description**_: iterates the set of keys. Please read how it works from Redis [scan](http://redis.io/commands/scan) command

##### *Parameters*
*number*: connection  
*number*: the cursor  
*array*:  for to hold the results  
*string (optional)*: for to `match` a given glob-style pattern, similarly to the behavior of the `keys` function that takes a pattern as only argument

##### *Return value*
`1` on success,  or `0` on the last iteration (when the returned cursor is equal 0). Returns `-1` on error. 


##### *Example*
{:lang="awk"}
    @load "redis"
    BEGIN{
     c=redis_connect()
     num=0
     while(1){
       ret=redis_scan(c,num,AR,"s*") # the last parameter (the pattern "s*"), is optional
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


### ttl, pttl
_**Description**_: Returns the time to live left for a given key in seconds (ttl), or milliseconds (pttl).

##### *Parameters*
*number*: connection  
*string*: key name  

##### *Return value*
*number*:  The time to live in seconds.  If the key has no ttl, `-1` will be returned, and `-2` if the key doesn't exist.

##### *Example*
{:lang="awk"}
    redis_ttl(c,"key")

### persist
_**Description**_: Remove the expiration timer from a key.

##### *Parameters*
*number*: connection  
*string*: key name  

##### *Return value*
`1` if a timeout was removed, `0` if key does not exist or does not have an associated timeout

##### *Example*
{:lang="awk"}
    redis_exists(c,"key)    # return 1
    redis_ttl(c,"key")      # returns -1 if has no associated expire
    redis_expire(c,"key",100)  # returns 1
    redis_persist(c,"key")     # returns 1
    redis_persist(c,"key")     # returns 0

### dump
_**Description**_: Dump a key out of a redis database, the value of which can later be passed into redis using the RESTORE command.  The data that comes out of DUMP is a binary representation of the key as Redis stores it.

##### *Parameters*
*number*: connection  
*string*: key name  

##### *Return value*
The Redis encoded value of the key, or `string null` if the key doesn't exist

##### *Examples*
{:lang="awk"}
    redis_set(c,"foo","bar")
    val=redis_dump(c,"foo")  # val will be the Redis encoded key value

### restore
_**Description**_: Restore a key from the result of a DUMP operation.

##### *Parameters*
*number*: connection  
*string*: key name.  
*number*: ttl number. How long the key should live (if zero, no expire will be set on the key).  
*string*: value string (binary).  The Redis encoded key value (from DUMP).

##### *Return value*
`1` on sucess, `-1` on error

##### *Examples*
{:lang="awk"}
    redis_set(c,"foo","bar")
    val=redis_dump(c,"foo")
    redis_restore(c,"bar",0,val)  # The key "bar", will now be equal to the key "foo"


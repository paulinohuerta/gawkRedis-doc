# Keys Functions {#keys}

1. [del](#del) - Delete a key
1. [dump](#dump) - Return a serialized version of the value stored at the specified key.
1. [exists](#exists) - Determine if a key exists
1. [expire, pexpire](#expire-pexpire) - Set a key's time to live in seconds
1. [keys](#keysfunction) - Find all keys matching the given pattern
1. [move](#move) - Move a key to another database
1. [object](#object) - Allows to inspect the internals of Redis Objects 
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
1. [restore](#restore) - Create a key using the provided serialized value, previously obtained with *dump*.

-----

### del {#del}
_**Description**_: Remove specified keys.

_**Parameters**_  
*string or array of string*: `key name` or `array name` containing the names of the keys

_**Return value**_    
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

### exists {#exists}
_**Description**_: Verify if the specified key exists.

_**Parameters**_     
*number*: connection  
*string*: key name

_**Return value**_    
`1` If the key exists, `0` if the key no exists.

{title="Example: Using exists",lang=text,linenos=off}
    redis_set(c,"key","value");
    redis_exists(c,"key"); # return 1
    redis_exists(c,"NonExistingKey") # return 0

### randomKey {#randomkey}
_**Description**_: Returns a random key.

_**Parameters**_    
*number*: connection  

_**Return value**_    
*string*: a random key from the currently selected database

{title="Example: Using randomKey",lang=text,linenos=off}
    print redis_randomkey(c)

### move {#move}
_**Description**_: Moves a key to a different database. The key will move only if not exists in destination database.

_**Parameters**_   
*number*: connection  
*string*: key, the key to move   
*number*: dbindex, the database number to move the key to  

_**Return value**_   
`1` if key was moved, `0` if key was not moved.

{title="Example: Using move",lang=text,linenos=off}
    redis_select(c,0)	  # switch to DB 0
    redis_set(c,"x","42") # write 42 to x
    redis_move(c,"x", 1)  # move to DB 1
    redis_select(c,1)	  # switch to DB 1
    redis_get(c,"x");	  # will return 42

### object {#object}
_**Description**_: allows to inspect the internals of Redis Objects associated with keys. It is useful for debugging or to understand if your keys are using the specially encoded data types to save space. Supports the sub commands: refcount, encoding and idletime.
You can to read more about the [`object` command](http://redis.io/commands/object)

_**Parameters**_   
*number*: connection  
*string*: sub command
*string*: key

_**Return value**_   
`number integers` for subcommands refcount and idletime.   
`string` for subcommand encoding.        
`null string` if the object to inspect is missing. 
`-1` when the subcommand is non-existent.

{title="Example: Using object",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      # print "type key students:433:",
            # redis_type(c,"students:433")
      # print "type key foo:",
            # redis_type(c,"foo")
      print "students:433 idletime:",
             redis_object(c,"idletime","students:433")
      print "foo idletime:",
            redis_object(c,"idletime","foo")
      # cob:11 can not exist
      if((value=redis_object(c,"idletime","cob:11"))=="")
        print "Key cob:11 non-existent"
      else
        print value 
      if((value=redis_object(c,"refcount","cob:11"))=="")
        print "Key cob:11 non-existent"
      else
        print value 
      print "foo refcount:",
            redis_object(c,"refcount","foo")
      print "foo encoding:",
            redis_object(c,"encoding","foo")
      print "students:433 refcount:",
            redis_object(c,"refcount","students:433")
      print "students:433 encoding:",
            redis_object(c,"encoding","students:433")
      # "command"  is not one of the three sub commands
      ret=redis_object(c,"command","students:433")
      if(ret==-1)
         print ERRNO
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    students:433 idletime: 263
    foo idletime: 263
    Key cob:11 non-existent
    Key cob:11 non-existent
    foo refcount: 1
    foo encoding: embstr
    students:433 refcount: 1
    students:433 encoding: ziplist
    object need a valid command refcount|encoding|idletime

### rename {#rename}
_**Description**_: Renames a key. If newkey already exists it is overwritten.

_**Parameters**_    
*number*: connection  
*string*: srckey, the key to rename.   
*string*: dstkey, the new name for the key.  

_**Return value**_    
`1` in case of success, `-1` in case of error.

{title="Example: Using rename",lang=text,linenos=off}
    redis_set(c,"x", "valx");
    redis_rename(c,"x","y");
    redis_get(c,"y")  # return "valx"
    redis_get(c,"x")  # return null string, because x 
                      # no longer exists

### renamenx {#renamenx}
_**Description**_: Same as rename, but will not replace a key if the destination already exists. This is the same behaviour as set and option nx.

_**Return value**_     
`1` in case of success, `0` in case not success.

### expire, pexpire {#expire-pexpire}
_**Description**_: Sets an expiration date (a timeout) on an item. pexpire requires a TTL in milliseconds.

_**Parameters**_    
*number*: connection  
*string*: key name. The key that will disappear.  
*number*: ttl. The key's remaining Time To Live, in seconds.

_**Return value**_    
`1` in case of success, `0` if key does not exist or the timeout could not be set

{title="Example: Using expire",lang=text,linenos=off}
    ret=redis_set(c,"x", "42") # ret value 1; x value "42"
    redis_expire(c,"x", 3) # x will disappear in 3 seconds
    system("sleep 5") # wait 5 seconds
    redis_get(c,"x") # will return null string,
                     # as x has expired

### keys {#keysfunction}
_**Description**_: Returns the keys that match a certain pattern. Check supported [glob-style patterns](http://redis.io/commands/keys)

_**Parameters**_    
*number*: connection  
*string*: pattern  
*array of strings*: the results, the keys that match a certain pattern.

_**Return value**_    
`1` in case of success, `-1` on error

{title="Example: Using keys",lang=text,linenos=off}
    redis_keys(c,"*",AR)   # all keys will match this
    delete AR
    # for matching all keys begining with "user"
    redis_keys(c,"user*",AR) 
    # show AR contains
    for(i in AR) {
      print i": "AR[i]
    }

### type {#type}
_**Description**_: Returns the type of data pointed by a given key.

_**Parameters**_    
*number*: connection  
*string*: key name  

_**Return value**_    
*string*: the type of the data (string, list, set, zset and hash) or `none` when the key does not exist.

{title="Example: Using type",lang=text,linenos=off}
    redis_set(c,"keyZ","valZ")
    ret=redis_type(c,"keyZ") # ret contains "string"
    # showing the "type" all keys of DB 4
    redis_select(c,4)
    redis_keys(c,"*",KEYS) 
    for(i in KEYS){
      print i": "KEYS[i]" ---> "redis_type(c,KEYS[i])
    } 

### sort {#sort}
_**Description**_: Sort the elements in a list, set or sorted set.

_**Parameters**_   
*number*: connection  
*string*: key name  
*array*: the array with the result  
*optional string*: options "desc|asc alpha"

_**Return value**_    
`1` or `-1`on error 

{title="Example: Using sort",lang=text,linenos=off}
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


### sortLimit {#sortlimit}
_**Description**_: Sort the elements in a list, set or sorted set, using the LIMIT modifier with the sense of limit the number of returned elements.

_**Parameters**_     
*number*: connection  
*string*: key name  
*array*: the array with the result 
*number*: offset
*number*: count
*optional string*: options "desc|asc alpha"

_**Return value**_    
`1` or `-1`on error 

{title="Example: Using type",lang=text,linenos=off}
    # will return 5 elements of the sorted version of
    #   list2, starting at element 0
    c=redis_connect()
    # assume "list2" with numerical content
    ret=redis_sortLimit(c,"list2",AR,0,5)
    # or using a sixth argument
    # ret=redis_sortLimit(c,"list2",AR,0,5,"desc") 
    #   for Alphanumeric content should use "alpha"
    if(ret==-1) {
      print ERRNO
    }
    for(i in AR){
      print i") "AR[i]
    }
    redis_close(c)


### sortLimitStore {#sortlimitstore}
_**Description**_: Sort the elements in a list, set or sorted set, using the LIMIT and STORE modifiers with the sense of limit the number of returned elements and ensure that the result is stored as in a new key instead of be returned.

_**Parameters**_   
*number*: connection  
*string*: key name  
*string*: the name of the new key 
*number*: offset
*number*: count
*otional string*: options "desc|asc alpha"

_**Return value**_   
`1` or `-1`on error 

{title="Example: Using sortLimitStore",lang=text,linenos=off}
    # will store 5 elements, of the sorted version of list2
    # in the list "listb"
    c=redis_connect()
    # assume "list2" with numerical content
    ret=redis_sortLimitStore(c,"list2","listb",0,5)
    # or using a sixth argument 
    # redis_sortLimitStore(c,"list2","listb",0,5,"desc")

### sortStore {#sortstore}
_**Description**_: Sort the elements in a list, set or sorted set, using the STORE modifier for that the result to be stored in a new key

##### *Parameters**_    
*number*: connection  
*string*: key name  
*string*: the name of the new key 
*optional string*: options "desc|asc alpha"

##### *Return value**_    
`1` or `-1`on error 

{title="Example: Using sortStore",lang=text,linenos=off}
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


### scan {#scan}
_**Description**_: iterates the set of keys. Please read how it works from Redis [scan](http://redis.io/commands/scan) command

_**Parameters**_    
*number*: connection  
*number*: the cursor  
*array*:  for to hold the results  
*string (optional)*: for to `match` a given glob-style pattern, similarly to the behavior of the `keys` function that takes a pattern as only argument

_**Return value**_   
`1` on success,  or `0` on the last iteration (when the returned cursor is equal 0). Returns `-1` on error. 


{title="Example: Using scan",lang=text,linenos=off}
    @load "redis"
    BEGIN{
     c=redis_connect()
     num=0
     while(1){
       # the last parameter (the pattern "s*"), is optional
       ret=redis_scan(c,num,AR,"s*") 
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

### ttl, pttl {#ttl-pttl}
_**Description**_: Returns the time to live left for a given key in seconds (ttl), or milliseconds (pttl).

_**Parameters**_    
*number*: connection  
*string*: key name  

_**Return value**_    
*number*:  The time to live in seconds.  If the key has no ttl, `-1` will be returned, and `-2` if the key doesn't exist.

{title="Example: Using ttl",lang=text,linenos=off}
    redis_ttl(c,"key")

### persist {#persist}
_**Description**_: Remove the expiration timer from a key.

_**Parameters**_   
*number*: connection  
*string*: key name  

_**Return value**_        
`1` if a timeout was removed, `0` if key does not exist or does not have an associated timeout

{title="Example: Using persist",lang=text,linenos=off}
    redis_exists(c,"key) # returns 1
    # returns -1 if has no associated expire
    redis_ttl(c,"key")
    redis_expire(c,"key",100)  # returns 1
    redis_persist(c,"key")     # returns 1
    redis_persist(c,"key")     # returns 0

### dump {#dump}
_**Description**_: Dump a key out of a redis database, the value of which can later be passed into redis using the RESTORE command.  The data that comes out of DUMP is a binary representation of the key as Redis stores it.

_**Parameters**_   
*number*: connection  
*string*: key name  

_**Return value**_        
The Redis encoded value of the key, or `string null` if the key doesn't exist

{title="Example: Using dump",lang=text,linenos=off}
    redis_set(c,"foo","bar")
    val=redis_dump(c,"foo")  # val will be the Redis encoded key value

### restore {#restore}
_**Description**_: Restore a key from the result of a DUMP operation.

_**Parameters**_   
*number*: connection  
*string*: key name.  
*number*: ttl number. How long the key should live (if zero, no expire will be set on the key).  
*string*: value string (binary).  The Redis encoded key value (from DUMP).

_**Return value**_        
`1` on sucess, `-1` on error

{title="Example: Using restore",lang=text,linenos=off}
    redis_set(c,"foo","bar")
    val=redis_dump(c,"foo")
    # The key "bar", will now be equal to the key "foo"
    redis_restore(c,"bar",0,val) 


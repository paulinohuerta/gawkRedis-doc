# Transactions {#transactions}
Recommended reading [Redis Transactions topic](http://redis.io/topics/transactions)

* [exec](#exec) - Executes all previously queued commands in a transaction and restores the connection state to normal.
* [multi](#multi) - Marks the start of a transaction block 
* [watch](#watch) - Marks the given keys to be watched for conditional execution of a transaction
* [discard](#discard) - Flushes all previously queued commands in a transaction
* [unwatch](#unwatch) - Flushes all the previously watched keys for a transaction

### multi {#multi}
_**Description**_:  Marks the start of a transaction block

_**Parameters**_    
*number*: connection  
 
_**Return value**_    
*number*: `1` always.

{title="Example: Using multi",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_multi(c)
      print redis_set(c,"SK1","valSK1")
      print redis_lrange(c,"list1",B,0,-1)
      print redis_llen(c,"list2")
      redis_exec(c,R)
       # do somthing with array R
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    QUEUED
    QUEUED
    QUEUED

### exec {#exec}
_**Description**_: Executes all previously queued commands in a transaction and restores the connection state to normal.

_**Parameters**_    
*number*: connection  
*array*: for the results. Each element being the reply to each of the commands in the atomic transaction  
 
_**Return value**_    
*number*: `1` on success, `0` if the execution was aborted (when using WATCH).

{title="Example: Using exec",lang=text,linenos=off}
    redis_exec(c,R)

### watch {#watch}
_**Description**_: Marks the given keys to be watched for conditional execution of a transaction. 

_**Parameters**_    
*number*: connection  
*string or array*: a key name or an array containing the key names
 
_**Return value**_    
*number*: always `1`

{title="Example: Using watch",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      AR[1]="list1"
      AR[2]="list2"
      redis_del(c,"list1")
      redis_del(c,"list2")
      LVAL[1]="one";
      LVAL[2]="two";
      LVAL[3]="three";
      redis_lpush(c,"list1",LVAL)
      redis_watch(c,AR)
      redis_multi(c)
      redis_set(c,"SK1","valSK1")
      redis_lrange(c,"list1",B,0,-1)
      redis_llen(c,"list2")
      redis_exec(c,R)
      print R[1]
      print R[2][1]"  "R[2][2]"  "R[2][3]
      print R[3]
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1
    three  two  one
    0

### unwatch {#unwatch}
_**Description**_: Flushes all the previously watched keys for a transaction. No need to use when was used EXEC or DISCARD

_**Parameters**_    
*number*: connection  
 
_**Return value**_    
*number*: always `1`

{title="Example: Using unwatch",lang=text,linenos=off}
    redis_unwatch(c)

### discard {#discard}
_**Description**_: Flushes all previously queued commands in a transaction and restores the connection state to normal. Unwatches all keys, if WATCH was used. 

_**Parameters**_    
*number*: connection  
 
_**Return value**_    
*number*: always `1`

{title="Example: Using discard",lang=text,linenos=off}
    redis_discard(c)

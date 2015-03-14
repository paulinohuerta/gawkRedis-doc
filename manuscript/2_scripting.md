# Scripting {#scripting}
Recommended reading [Redis Lua scripting](http://redis.io/commands/eval)

* [evalRedis](#evalredis) - Executes a Lua script server side
* [evalsha](#evalsha) - Executes a Lua script server side. The script had must been cached previously
* [script exists](#script-exists) - Checks existence of scripts in the scripts cache
* [script flush](#script-flush) - Removes all the scripts from the scripts cache
* [script kill](#script-kill) - Kills the script currently in execution
* [script load](#script-load) - Loads the specified Lua script into the scripts cache

### evalRedis {#evalredis}
_**Description**_:  Evaluates scripts using the Lua interpreter built into Redis.

_**Parameters**_    
*number*: connection  
*string*: the Lua script   
*number*: the number of arguments, that represent Redis key names   
*array*: containing the arguments  
*array*: to store the results, but it not be always will contain results (read the example). Also this array may contain subarrays   
 
_**Return value**_    
*number* or *string*: `1` when it puts the results in the arrray. `-1` on error: `NOSCRIPT` No matching script.

{title="Example: Using evalRedis",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      ARG[1]="hset"
      ARG[2]="thehash"
      ARG[3]="field3"
      ARG[4]="value3"
      ret=redis_evalRedis(c,"return redis.call(KEYS[1],ARGV[1],ARGV[2],ARGV[3])",1,ARG,R)
      print "Function 'evalRedis' returns: "ret
      print "Elements in arrray of results: "length(R)
      print redis_hget(c,"thehash","field3")
      redis_close(c)
    } 

{title="Output",lang=text,linenos=off}
    Function 'evalRedis' returns: 1
    Elements in arrray of results: 0
    value3

### script exists {#script-exists}
_**Description**_: Returns information about the existence of the scripts in the script cache.  Accepts one or more SHA1 digests.
For detailed information about [Redis Lua scripting](http://redis.io/commands/eval)

_**Parameters**_    
*number*: connection handle  
*string*: "exists"
*array*: containing the SHA1 digests  
*array*: an array of integers that correspond to the specified SHA1 digest. It stores `1` for a script that actually exists in the script cache, otherwise `0` is stored.

_**Return value**_    
*number*: `1` on success, `0` if array of SHA1 digests (third argument) is empty. `-1` on error.   

{title="Example: Using script exists",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
       #  'script load' returns SHA1 digest if success
      A[1]=redis_script(c,"load","return {1,2,{7,'Hello World!',89}}")
      A[2]=redis_script(c,"load","return redis.call('set','foo','bar')")
      A[3]=redis_script(c,"load","return redis.call(KEYS[1],ARGV[1])")
      ret=redis_script(c,"exists",A,R)
      print "Obtain information of existence for these",
            "three scripts whose keys are:"
      for(i in A) {
       print A[i]
      }
      print "script exists returns: "ret
      print "The results of command are:"
      for(i in R) {
        print i") "R[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    Obtain information of existence for these three scripts whose keys are:
    4647a689ee8af8debe9fd50a6fb9fee93ef92e43
    2fa2b029f72572e803ff55a09b1282699aecae6a
    24598a5b88e25cb396a4de4afbd1f5509c537396
    script exists returns: 1
    The results of command are:
    1) 1
    2) 1
    3) 1

### script load {#script-load}
_**Description**_: Loads a script into the scripts cache, without executing it.
For detailed information about [Redis Lua scripting](http://redis.io/commands/eval)

_**Parameters**_    
*number*: connection handle  
*string*: "load"   
*string*: the Lua script  

_**Return value**_    
*string*: returns the SHA1 digest of the script added into the script cache 

{title="Example: Using script load",lang=text,linenos=off}
    c=redis_connect()
    k1=redis_script(c,"load","return redis.call('set','foo','bar')")
     # 'k1' stores the SHA1 digest

### script kill {#script-kill}
_**Description**_: Kills the currently executing Lua script
For detailed information about [Redis Lua scripting](http://redis.io/commands/eval)

_**Parameters**_    
*number*: connection handle  
*string*: "kill"   

_**Return value**_    
*number*: `1` on sucess, `-1` on error, by example: NOTBUSY No scripts in execution right now.

{title="Example: Using script kill",lang=text,linenos=off}
    c=redis_connect()
    redis_script(c,"kill")

### script flush {#script-flush}
_**Description**_: Flush the Lua scripts cache
For detailed information about [Redis Lua scripting](http://redis.io/commands/eval)

_**Parameters**_    
*number*: connection handle  
*string*: "flush"   

_**Return value**_    
*number*: `1` on success 

{title="Example: Using script flush",lang=text,linenos=off}
    c=redis_connect()
    redis_script(c,"flush")

### evalsha {#evalsha}
_**Description**_:  evalsha works exactly like evalRedis, but instead of having a script as the first argument it has the SHA1 digest of a script.

_**Parameters**_    
*number*: connection  
*string*: the SHA1 digest of a script   
*number*: the number of arguments, that represent Redis key names  
*array*: containing the arguments  
*array*: to store the results, but it not be always will contain results (read the example). Also this array may contain subarrays 
 
_**Return value**_    
*number* or *string*: `1` when it puts the results in the arrray. `-1` on error: `NOSCRIPT` No matching script.

{title="Example: Using evalsha",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
       #  loading into the scripts cache
      cmd1=redis_script(c,"load","return {1,2,{7,'Hello World!',89}}")
      cmd2=redis_script(c,"load","return redis.call('set','foo','bar')")
      cmd3=redis_script(c,"load","return redis.call(KEYS[1],ARGV[1])")
       #  executing the scripts
      print "Returns cmd1:",
            redis_evalsha(c,cmd1,0,ARG,R)
      print "Elements in arrray R (the results): "length(R)
       # Elements in R are
       # R[1], R[2], R[3][1], R[3][2], R[3][3]
      delete R
      print "Returns cmd2:",
            redis_evalsha(c,cmd2,0,ARG,R)
      print "Elements in arrray R (the results): "length(R)
       # the arguments for the next
      ARG[1]="hvals"
      ARG[2]="thehash"
      print "Returns cmd3: "redis_evalsha(c,cmd3,1,ARG,R)
      print "Elements in arrray R (the results): "length(R)
       # Compare the return value of the next command
      ARG[1]="type"
      delete R
      print "Now cmd3 returns a string:",
             redis_evalsha(c,cmd3,1,ARG,R)
      print "Elements in arrray R (the results): "length(R)
      redis_close(c)
    } 

{title="Output",lang=text,linenos=off}
    Returns cmd1: 1
    Elements in arrray R (the results): 3
    Returns cmd2: 1
    Elements in arrray R (the results): 0
    Returns cmd3: 1
    Elements in arrray R (the results): 30
    Now cmd3 returns a string: hash
    Elements in arrray R (the results): 0
    

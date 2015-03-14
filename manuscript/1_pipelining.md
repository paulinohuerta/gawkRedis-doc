# Pipelining {#pipelining}
Recommended reading for to know as this is supported: [Redis pipelining](http://redis.io/topics/pipelining) and [hiredis pipelining](https://github.com/redis/hiredis#pipelining), who works in a more low layer.

* [pipeline](#pipeline) - To create a pipeline, allowing buffered commands
* [getReply](#getreply) - To get or receive the result of each command buffered

### pipeline {#pipeline}
_**Description**_:  To create a pipeline, allowing buffered commands.

_**Parameters**_     
*number*: connection  

_**Return value**_
*number*: `pipe handle` on success, `-1` on error

{title="Example: Using pipeline",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      p=redis_pipeline(c)  # 'p' is a new pipeline
       # The following SET commands are buffered
      redis_select(p,4) # changing db, using the  command select
      redis_set(p,"newKey","newValue") # set command
      redis_type(p,"newKey") # type command
      redis_setrange(p,"newKey",6,"123") # setrange command
      redis_dump(p,"newKey") # dump command
      redis_keys(p,"n*",AR) # keys command
       # To execute all commands buffered, and store the return values
      for( ; ERRNO=="" ; RET[++i]=redis_getReply(p,REPLY) ) ;
      ERRNO=""
       # To use the value returned by 'redis_dump'
      redis_restore(c,"newKey1","0",RET[5])
       # Actually the array REPLY stores the result the last command buffered
       # Then, for know the result of 'redis_keys':
      for( j in REPLY ) {
        print j": "REPLY[j]
      }
      redis_close(c)
    }

### getReply {#getreply}
_**Description**_: To receive the replies, the first time sends all buffered commands to the server, then subsequent calls get replies for each command.

_**Parameters**_     
*number*: pipeline handle  
*array*: for results. Will be used or no, according to command in question

_**Return value**_
*string or number*: the return value of the following command in the buffer,  `-1` on error (if not exist results buffered)

{title="Example: Using getReply",lang=text,linenos=off}
    c=redis_connect()
    p=redis_pipeline(c)
    redis_hset(p,"thehash","field1","25")
    redis_hset(p,"thehash","field2","26")
     # To execute all and obtain the return of the first
    r1=redis_getReply(p,REPLY)
     # To get the reply of second 'hset'
    r2=redis_getReply(p,REPLY)
    print r1,r2
     # Now there are no results in the buffer, and
     #  using 'the pipeline handle' can be reused,
     #  no need to close the pipeline once completed their use


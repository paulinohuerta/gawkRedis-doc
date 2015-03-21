# Pub/sub Functions {#pubsub}
Recommended reading about the paradigm [Pub/Sub](http://redis.io/topics/pubsub) and the implemetation

* [publish](#publish) - Post a message to the given channel
* [subscribe](#subscribe) - Subscribes the client to the specified channels.
* [psubscribe](#psubscribe) - Subscribes the client to the given patterns. Supported glob-style patterns.
* [unsubscribe](#unsubscribe) - Unsubscribes the client from the given channels, or from all of them if none is given.
* [punsubscribe](#punsubscribe) - Unsubscribes the client from the given patterns, or from all of them if none is given.
* [getMessage](#getmessage) - Way in which a subscriber consumes a message 

### publish {#publish}
_**Description**_: Publish messages to channels.

_**Parameters**_   
*number*: connection  
*string*: a channel to publish to  
*string*: a string messsage  

_**Return value**_   
*number*: the number of clients that received the message

{title="Example: Using publish",lang=text,linenos=off}
    redis_publish(c,"chan-1", "hello, world!")
    # send message

### subscribe {#subscribe}
_**Description**_: Subscribe to channels.

_**Parameters**_   
*number*: connection  
*string or array*: the channel name or the array containing the names of channels  

_**Return value**_   
`1` on success, `-1` on error

{title="Example: Using subscribe",lang=text,linenos=off}
    redis_subscribe(c,"chan-2")  # returns 1,
    # subscribes to chan-2
    #
    CH[1]="chan-1"
    CH[2]="chan-2"
    CH[3]="chan-3"
    #
    redis_subscribe(c,CH)  # returns 1,
    # subscribes to chan-1, chan-2 and chan-3

### unsubscribe {#unsubscribe}
_**Description**_: Unsubscribes the client from the given channels, or from all of them if none is given.

_**Parameters**_   
*number*: connection  
*string or array (This parameter could not be )*: the channel name or the array containing the names of channels  

_**Return value**_   
`1` on success, `-1` on error

{title="Example: Using unsubscribe",lang=text,linenos=off}
    redis_unsubscribe(c,"chan-2")
     # returns 1, unsubscribes to chan-2
    CH[1]="chan-1"; CH[2]="chan-2"; CH[3]="chan-3"
     # unsubscribes to chan-1, chan-2 and chan-3
    redis_unsubscribe(c,CH)  # returns 1
    # unsubscribing from all the previously
    # subscribed channels

### punsubscribe {#punsubscribe}
_**Description**_: Unsubscribes the client from the given patterns, or from all of them if none is given.

_**Parameters**_   
*number*: connection  
*string or array (This parameter could not be )*: the pattern or the array containing the patterns.   
_**Return value**_   
`1` on success, `-1` on error

{title="Example: Using punsubscribe",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      redis_psubscribe(c,"ib*")
      redis_subscribe(c,"channel1")
      while(ret=redis_getMessage(c,A)) {
        for(i in A) {
          print i": "A[i]
        }
        if(A[4]=="exit" && A[3]=="ibi") {
          redis_punsubscribe(c,"ib*")
        }
        if(A[3]=="exit" && A[2]=="channel1") {
          break
        }
        delete A
      }
      redis_unsubscribe(c)
      redis_close(c)
    }

### psubscribe {#psubscribe}
_**Description**_: Subscribes the client to the given patterns. Supported glob-style patterns.

_**Parameters**_   
*number*: connection  
*string or array*: the pattern, or the array containing the patterns.

_**Return value**_   
`1` on success, `-1` on error

{title="Example: Using psubscribe",lang=text,linenos=off}
    # subscribes to channels that match the pattern 'ib'
    # to the begin of the name
    redis_psubscribe(c,"ib*")  # returns 1 
    CH[1]="chan[ae]-1"
    CH[2]="chan[ae]-2"
    redis_psubscribe(c,CH)  # returns 1,
    # subscribes to chana-1, chane-1, chana-2, chane-2

### getMessage {#getmessage}
_**Description**_: Gets a message from any of the subscribed channels, (based at hiredis API redisGetReply for to consume messages).

_**Parameters**_   
*number*: connection  
*array*: containing the messages received  

_**Return value**_   
`1` on success, `-1` on error

{title="Example: Using getMessage",lang=text,linenos=off}
    A[1]="c1"
    A[2]="c2"
    ret=redis_subscribe(c,A)
    while(ret=redis_getMessage(c,B)) {
       for(i in B){
         print i") "B[i]
       }
       delete B
    }


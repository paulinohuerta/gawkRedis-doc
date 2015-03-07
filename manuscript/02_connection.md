# Connection Functions

1. [connect](#connect) - Connect to a Redis server
1. [auth](#auth) - Authenticate to the server
1. [select](#select) - Change the selected database for the current connection
1. [close, disconnect](#close-disconnect) - Close the connection
1. [ping](#ping) - Ping the server
1. [echo](#echo) - Echo the given string

### connect     
_**Description**_: Connects to a Redis instance.    

_**Parameters**_   
*host*: string, optional  
*port*: number, optional    
_**Return value**_     
*connection handle*: number, `-1` on error.

##### *Example*    
{lang=awk,line-numbers=off}
    c=redis_connect('127.0.0.1', 6379)
    # port 6379 by default
    c=redis_connect('127.0.0.1')
    # host address 127.0.0.1 and port 6379 by default
    c=redis_connect()

### auth    
_**Description**_: Authenticate the connection using a password.   

##### *Parameters*
*number*: connection    
*string*: password    

##### *Return value* 
`1` if the connection is authenticated, `null string` (empty string) otherwise.    
##### *Example*    
{:lang="awk"}
    ret=redis_auth(c,"fooXX")
    if(ret) {
      # authenticated
    }
    else {
      # not authenticated
    }

### select
_**Description**_: Change the selected database for the current connection.

##### *Parameters*
*number*: dbindex, the database number to switch to

##### *Return value*
`1` in case of success, `-1` in case of failure.

##### *Example*
{:lang="awk"}
    redis_select(c,5)

### close, disconnect
_**Description**_: Disconnects from the Redis instance.

##### *Parameters*
*number*: connection handle  

##### *Return value*
`1` on success, `-1` on error.

##### *Example*
{:lang="awk"}
    ret=redis_close(c)
    if(ret==-1) {
      print ERRNO
    }

### ping
_**Description**_: Check the current connection status

##### *Parameters*
*number*: connection handle  

##### *Return value*
*string*: `PONG` on success.

### echo
_**Description**_: Sends a string to Redis, which replies with the same string

##### *Parameters*
*number*: connection
*string*: The message to send.

##### *Return value*
*string*: the same message.


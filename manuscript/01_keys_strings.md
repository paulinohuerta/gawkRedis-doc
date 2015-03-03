# gawkRedis

A [GAWK](https://www.gnu.org/software/gawk/) (the GNU implementation of the AWK Programming Language) client library for Redis.

The gawk-redis is an extension library that enables GAWK , to process data from a [Redis server](http://redis.io/), then provides an API for communicating with the Redis key-value store, using [hiredis](https://github.com/redis/hiredis), a C client for Redis.

The prefix "redis_" must be at the beginning of each function name, as shown in the code examples, although the explanations are omitted for clarity.

# Installing/Configuring

Everything you should need to install gawk-redis on your system.

## Installation

* Install [hiredis](https://github.com/redis/hiredis), library C client for Redis.

* The README file will explain how to build the Redis extensions for gawk.

* Interested in release candidates or unstable versions? [check the repository](https://sourceforge.net/u/paulinohuerta/gawkextlib_d/ci/master/tree/)

 You can try running the following gawk script, *myscript.awk*, which uses the extension:

{lang="AWK"}
    @load "redis"
    BEGIN{
      # the connection with the server: 127.0.0.1:6379
      c=redis_connect()
      if(c==-1) {
        # always you can to use the ERRNO var for checking
        print ERRNO
      }
      # the select redis command
      ret=redis_select(c,4) 
      # The above statement assumes that db 4 contains data
      print "select returns "ret
      pong=redis_ping(c) # the ping redis command
      print "The server says: "pong
       # the echo redis command
      print redis_echo(c,"foobared")
      redis_close(c)
    }

which must run with:

    /path-to-gawk/gawk -f myscript.awk /dev/null


# The API Functions

* [Connection](#connection)
* [Keys and strings](#keys-and-strings)
* [Hashes](#hashes)
* [Lists](#lists)
* [Sets](#sets)
* [Sorted sets](#sorted-sets)
* [Pub/sub](#pubsub) 
* [Pipelining](#pipelining)
* [Scripting](#scripting)
* [Server](#server)  
* [Transactions](#transactions)

# Connection Functions

1. [connect](#connect) - Connect to a Redis server
1. [auth](#auth) - Authenticate to the server
1. [select](#select) - Change the selected database for the current connection
1. [close, disconnect](#close-disconnect) - Close the connection
1. [ping](#ping) - Ping the server
1. [echo](#echo) - Echo the given string

### connect     
_**Description**_: Connects to a Redis instance.    

##### *Parameters*    
*host*: string, optional  
*port*: number, optional    

##### *Return value*    
*connection handle*: number, `-1` on error.

##### *Example*    
~~~ awk
    c=redis_connect('127.0.0.1', 6379)
    c=redis_connect('127.0.0.1') # port 6379 by default
    # host address 127.0.0.1 and port 6379 by default
    c=redis_connect()
    a=length($0)
    b=substring($1,3)
~~~

### auth    
_**Description**_: Authenticate the connection using a password.   

##### *Parameters*   
*number*: connection    
*string*: password    

##### *Return value*    
`1` if the connection is authenticated, `null string` (empty string) otherwise.   
##### *Example*    
{lang="Awk"}
    ret=redis_auth(c,"fooXX");
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
-----
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
-----
_**Description**_: Check the current connection status

##### *Parameters*
*number*: connection handle  

##### *Return value*
*string*: `PONG` on success.


### echo
-----
_**Description**_: Sends a string to Redis, which replies with the same string

##### *Parameters*
*number*: connection
*string*: The message to send.

##### *Return value*
*string*: the same message.


# Keys and Strings

## Strings

* [append](#append) - Append a value to a key
* [bitcount](#bitcount) - Count set bits in a string
* [bitop](#bitop) - Perform bitwise operations between strings
* [decr, decrby](#decr-decrby) - Decrement the value of a key
* [get](#get) - Get the value of a key
* [getbit](#getbit) - Returns the bit value at offset in the string value stored at key
* [getrange](#getrange) - Get a substring of the string stored at a key
* [getset](#getset) - Set the string value of a key and return its old value
* [incr, incrby](#incr-incrby) - Increment the value of a key
* [incrbyfloat](#incrbyfloat) - Increment the float value of a key by the given amount
* [mget](#mget) - Get the values of all the given keys
* [mset](#mset) - Set multiple keys to multiple values
* [set](#set) - Set the string value of a key
* [setbit](#setbit) - Sets or clears the bit at offset in the string value stored at key
* [setrange](#setrange) - Overwrite part of a string at key starting at the specified offset
* [strlen](#strlen) - Get the length of the value stored in a key

## Keys

* [del](#del) - Delete a key
* [dump](#dump) - Return a serialized version of the value stored at the specified key.
* [exists](#exists) - Determine if a key exists
* [expire, pexpire](#expire-pexpire) - Set a key's time to live in seconds
* [keys](#keys) - Find all keys matching the given pattern
* [move](#move) - Move a key to another database
* [persist](#persist) - Remove the expiration from a key
* [randomkey](#randomkey) - Return a random key from the keyspace
* [rename](#rename) - Rename a key
* [renamenx](#renamenx) - Rename a key, only if the new key does not exist
* [sort](#sort) - Sort the elements in a list, set or sorted set
* [sortLimit](#sortlimit) - Sort the elements in a list, set or sorted set, using the LIMIT modifier
* [sortLimitStore](#sortlimitstore) - Sort the elements in a list, set or sorted set, using the LIMIT and STORE modifiers
* [sortStore](#sortstore) - Sort the elements in a list, set or sorted set, using the STORE modifier
* [scan](#scan) - iterates the set of keys in the currently selected Redis db
* [type](#type) - Determine the type stored at key
* [ttl, pttl](#ttl-pttl) - Get the time to live for a key
* [restore](#restore) - Create a key using the provided serialized value, previously obtained with [dump](#dump).

-----

### get
-----
_**Description**_: Get the value related to the specified key

##### *Parameters*
*number*: connection  
*string*: the key

##### *Return value*
*string*: `key value` or `null string` (empty string) if key didn't exist.

##### *Example*
{:lang="awk"}
    value=redis_get(c,"key1")

### set
-----
_**Description**_: Set the string value in argument as value of the key.  If you're using Redis >= 2.6.12, you can pass extended options as explained below

##### *Parameters*
*number*: connection  
*string*: key  
*string*: value  
*and optionally*: "EX",timeout,"NX" or "EX",timeout,"XX" or "PX" instead of "EX"

##### *Return value*
`1` if the command is successful `string null` if no success, or `-1` on error.

##### *Example*
{:lang="awk"}
    # Simple key -> value set
    redis_set(c,"key","value");

    # Will redirect, and actually make an SETEX call
    redis_set(c,"mykey1","myvalue1","EX",10)

    # Will set the key, if it doesn't exist, with a ttl of 10 seconds
    redis_set(c,"mykey1","myvalue1","EX",10,"NX")

    # Will set a key, if it does exist, with a ttl of 10000 miliseconds
    redis_set(c,"mykey1","myvalue1","PX",10000,"XX")


### del
-----
_**Description**_: Remove specified keys.

##### *Parameters*
*string or array of string*: `key name` or `array name` containing the names of the keys

##### *Return value*
*number*: Number of keys deleted.

##### *Example*
{:lang="awk"}
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
-----
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

### incr, incrby
-----
_**Description**_: Increment the number stored at key by one. If the second argument is filled, it will be used as the integer value of the increment.

##### *Parameters*
*number*: connection  
*string*: key name  
*number*: value that will be added to key (only for incrby)

##### *Return value*
*number*: the new value

##### *Example*
{:lang="awk"}
    redis_incr(c,"key1") # key1 didn't exists, set to 0 before the increment
                   # and now has the value 1
    redis_incr(c,"key1") #  value 2
    redis_incr(c,"key1") #  value 3
    redis_incr(c,"key1") #  value 4
    redis_incrby(c,"key1",10) #  value 14

### incrbyfloat
-----
_**Description**_: Increment the key with floating point precision.

##### *Parameters*
*number*: connection  
*string*: key name  
*value*: (float) value that will be added to the key  

##### *Return value*
*number*: the new value

##### *Example*
{:lang="awk"}
    redis_incrbyfloat(c,"key1", 1.5)  # key1 didn't exist, so it will now be 1.5
    redis_incrbyfloat(c,"key1", 1.5)  # 3
    redis_incrbyfloat(c,"key1", -1.5) # 1.5
    redis_incrbyfloat(c,"key1", 2.5)  # 4

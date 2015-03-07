# Strings

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

-----

### get
_**Description**_: Get the value related to the specified key

##### *Parameters*
*number*: connection  
*string*: the key

##### *Return value*
*string*: `key value` or `null string` (empty string) if key didn't exist.

##### *Examples*
{:lang="awk"}
    value=redis_get(c,"key1")

### set
_**Description**_: Set the string value in argument as value of the key.  If you're using Redis >= 2.6.12, you can pass extended options as explained below

##### *Parameters*
*number*: connection  
*string*: key  
*string*: value  
*and optionally*: "EX",timeout,"NX" or "EX",timeout,"XX" or "PX" instead of "EX"

##### *Return value*
`1` if the command is successful `string null` if no success, or `-1` on error.

##### *Examples*
{:lang="awk"}
    # Simple key -> value set
    redis_set(c,"key","value");

    # Will redirect, and actually make an SETEX call
    redis_set(c,"mykey1","myvalue1","EX",10)

    # Will set the key, if it doesn't exist, with a ttl of 10 seconds
    redis_set(c,"mykey1","myvalue1","EX",10,"NX")

    # Will set a key, if it does exist, with a ttl of 10000 miliseconds
    redis_set(c,"mykey1","myvalue1","PX",10000,"XX")

### incr, incrby
_**Description**_: Increment the number stored at key by one. If the second argument is filled, it will be used as the integer value of the increment.

##### *Parameters*
*number*: connection  
*string*: key name  
*number*: value that will be added to key (only for incrby)

##### *Return value*
*number*: the new value

##### *Examples*
{:lang="awk"}
    redis_incr(c,"key1") # key1 didn't exists, set to 0 before the increment
                   # and now has the value 1
    redis_incr(c,"key1") #  value 2
    redis_incr(c,"key1") #  value 3
    redis_incr(c,"key1") #  value 4
    redis_incrby(c,"key1",10) #  value 14

### incrbyfloat
_**Description**_: Increment the key with floating point precision.

##### *Parameters*
*number*: connection  
*string*: key name  
*value*: (float) value that will be added to the key  

##### *Return value*
*number*: the new value

##### *Examples*
{:lang="awk"}
    redis_incrbyfloat(c,"key1", 1.5)  # key1 didn't exist, so it will now be 1.5
    redis_incrbyfloat(c,"key1", 1.5)  # 3
    redis_incrbyfloat(c,"key1", -1.5) # 1.5
    redis_incrbyfloat(c,"key1", 2.5)  # 4

### decr, decrby
_**Description**_: Decrement the number stored at key by one. If the second argument is filled, it will be used as the integer value of the decrement.

##### *Parameters*
*number*: connection  
*string*: key name  
*number*: value that will be substracted to key (only for decrby)

##### *Return value*
*number*: the new value

##### *Examples*
{:lang="awk"}
    redis_decr(c,"keyXY") # keyXY didn't exists, set to 0 before the increment 
                    # and now has the value -1
    redis_decr(c,"keyXY") # -2
    redis_decr(c,"keyXY")   # -3
    redis_decrby(c,"keyXY",10)  # -13

### mget
_**Description**_: Get the values of all the specified keys. If one or more keys dont exist, the array will contain `null string` at the position of the key.

##### *Parameters*
*number*: connection  
*Array*: Array containing the list of the keys  
*Array*: Array of results, containing the values related to keys in argument

##### *Return value*
`1` success `-1` on error

##### *Examples*
{:lang="awk"}
    @load "redis"
    BEGIN{
     null="\"\""
     c=redis_connect()
     redis_set(c,"keyA","val1")
     redis_set(c,"keyB","val2")
     redis_set(c,"keyC","val3")
     redis_set(c,"keyD","val4")
     redis_set(c,"keyE","")
     AR[1]="keyA"
     AR[2]="keyB"
     AR[3]="keyZ" # this key no exists
     AR[4]="keyC"
     AR[5]="keyD"
     AR[6]="keyE"
     ret=redis_mget(c,AR,K) # K is the array with results
     for(i=1; i<=length(K); i++){
       if(!K[i]) {
         if(redis_exists(c,AR[i])){ # function exists was described previously
           print i": "AR[i]" ----> "null
         }
         else {
           print i": "AR[i]" ----> not exists"
         }
       }
       else {
         print i": "AR[i]" ----> ""\""K[i]"\""
       }
     }
     redis_close(c)
    }

### getset
_**Description**_: Sets a value and returns the previous entry at that key.

##### *Parameters*
*number*: connection  
*string*: key name   
*string*: key value

##### *Return value*
A string, the previous value located at this key

##### *Example*
{:lang="awk"}
    redis_set(c,"x", "42")
    exValue=redis_getset(c,"x","lol") # return "42", now the value of x is "lol"
    newValue = redis_get(c,"x") # return "lol"

### append
_**Description**_: Append specified string to the string stored in specified key.

##### *Parameters*
*number*: connection  
*string*: key name   
*string*: value

##### *Return value*
*number*: Size of the value after the append

##### *Example*
{:lang="awk"}
    redis_set(c,"key","value1")
    redis_append(c,"key","value2") # 12 
    redis_get(c,"key") # "value1value2"

### getrange
_**Description**_: Return a substring of a larger string 

##### *Parameters*
*number*: connection  
*string*: key name  
*number*: start  
*number*: end  

##### *Return value*
*string*: the substring 

##### *Example*
{:lang="awk"}
    redis_set(c,"key","string value");
    print redis_getrange(c,"key", 0, 5)  # "string"
    print redis_getrange(c,"key", -5, -1)  # "value"

### setrange
_**Description**_: Changes a substring of a larger string.

##### *Parameters*
*number*: connection  
*string*: key name  
*number*: offset  
*string*: value

##### *Return value*
*string*: the length of the string after it was modified.

##### *Example*
{:lang="awk"}
    redis_set(c,"key1","Hello world")
    ret=redis_setrange(c,"key1",6,"redis") # ret value 11
    redis_get(c,"key1") # "Hello redis"


### strlen
_**Description**_: Get the length of a string value.

##### *Parameters*
*number*: connection  
*string*: key name  

##### *Return value*
*number*

##### *Example*
{:lang="awk"}
    redis_set(c,"key","value")
    redis_strlen(c,"key")  # 5


### getbit
_**Description**_: Return a single bit out of a larger string

##### *Parameters*
*number*: connection  
*string*: key name  
*number*: offset

##### *Return value*
*number*: the bit value (0 or 1)

##### *Example*
{:lang="awk"}
    redis_set(c,"key", "\x7f"); // this is 0111 1111
    redis_getbit(c,"key", 0) # 0
    redis_getbit(c,"key", 1) # 1
    redis_set(c,"key", "s"); // this is 0111 0011
    print redis_getbit(c,"key", 5) # 0
    print redis_getbit(c,"key", 6) # 1
    print redis_getbit(c,"key", 7) # 1

### setbit
_**Description**_: Changes a single bit of a string.

##### *Parameters*
*number*: connection  
*string*: key name   
*number*: offset  
*number*: value (1 or 0)

##### *Return value*
*number*: 0 or 1, the value of the bit before it was set.

##### *Example*
{:lang="awk"}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_set(c,"key", "*") # ord("*") = 42 = "0010 1010"
      redis_setbit(c,"key", 5, 1) #  returns 0
      redis_setbit(c,"key", 7, 1) # returns 0
      print redis_get(c,"key") #  "/" = "0010 1111"
      redis_set(c,"key1","?") # 00111111
      print redis_get(c,"key1")
      print "key1: changing bit 7, it returns "setbit(c,"key1", 7, 0) # returns 1
      print "key1: value actual is 00111110"
      print redis_get(c,"key1") # retorna ">"
      redis_close(c)
    }

### bitop
_**Description**_: Bitwise operation on multiple keys.

##### *Parameters*
*number*: connection  
*operator*: either "AND", "OR", "NOT", "XOR"   
*ret_key*: result key   
*array or string*: array containing the keys or only one string (in case of using the NOT operator).

##### *Return value*
*number*: The size of the string stored in the destination key.

### bitcount
_**Description**_: Count bits in a string.

##### *Parameters*
*number*: connection  
*string*: key name  

##### *Return value*
*number*: The number of bits set to 1 in the value behind the input key.

### mset, msetnx
_**Description**_: Sets multiple key-value pairs in one atomic command. msetnx only returns `1` if all the keys were set (see set and option nx).

##### *Parameters*
*number*: connection  
*array*: keys and their respectives values  

##### *Return value*
`1` in case of success, `-1` on error. while msetnx returns `0` if no key was set (at least one key already existed).

##### *Example*
{:lang="awk"}
    @load "redis"
    BEGIN {
     AR[1]="q1"
     AR[2]="vq1"
     AR[3]="q2"
     AR[4]="vq2"
     AR[5]="q3"
     AR[6]="vq3"
     AR[7]="q4"
     AR[8]="vq4"
     c=redis_connect()
     ret=redis_mset(c,AR)
     print ret" returned by mset"
     redis_keys(c,"q*",R)
     for(i in R){
       print i") "R[i]
     }
     redis_close(c)
    }

Output:

    1 returned by mset
    1) q2
    2) q3
    3) q4
    4) q1


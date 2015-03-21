# Lists Functions {#lists}

1. [lindex](#lindex) - Returns the element at index index in the list.
1. [linsertBefore](#linsertbefore) - Inserts value in a list key before the reference value pivot.
1. [linsertAfter](#linsertafter) - Inserts value in a list key after the reference value pivot.
1. [llen](#llen) - Gets the length/size of a list
1. [lpop](#lpop) - Remove and get the first element in a list
1. [lpush](#lpush) - Insert all the specified values at the head of a list
1. [lpushx](#lpushx) - Inserts a value at the head of the list, only if the key already exists and holds a list.
1. [lrange](#lrange) - Get a range of elements from a list
1. [lrem](#lrem) - Remove elements from a list
1. [lset](#lset) - Set the value of an element in a list by its index
1. [ltrim](#ltrim) - Trim a list to the specified range
1. [rpop](#rpop) - Remove and get the last element in a list
1. [rpoplpush](#rpoplpush) - Returns and removes the last element (tail) of a list, and pushes the element at the first element (head) of other list.
1. [rpush](#rpush) - Insert all the specified values at the tail of a list
1. [rpushx](#rpushx) - Inserts a value at the tail of the list, only if the key already exists and holds a list.
1. [blpop](#blpop) - Is a blocking list pop primitive. Pops elements from the head of a list.
1. [brpop](#brpop) - Is a blocking list pop primitive. Pops elements from the tail of a list.
1. [brpoplpush](#brpoplpush) - Is the blocking variant of RPOPLPUSH.

### lindex {#lindex}
_**Description**_: Returns the element at index index in the list.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: the index  

_**Return value**_   
*string*: the requested element, or `null string` when index is out of range. `-1` on error

{title="Example: Using lindex",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"mylist99")
      redis_lpush(c,"mylist99","s1")
      redis_lpush(c,"mylist99","s0")
       # gets element index 0
      print redis_lindex(c,"mylist99",0)
       # gets the last element
      print redis_lindex(c,"mylist99",-1)
       # gets null string
      print redis_lindex(c,"mylist99",3)
      redis_close(c)
    }

### linsertBefore {#linsertbefore}
_**Description**_: Inserts value in a list key before the reference value pivot.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: pivot 
*string*: value

_**Return value**_   
*number*: the length of the list after the insert, or `-1` when the value pivot was not found.

{title="Example: Using linsertBefore",lang=text,linenos=off}
    redis_del(c,"mylist")
    print redis_rpush(c,"mylist","Hello")
    print redis_rpush(c,"mylist","World")
    print redis_linsertBefore(c,"mylist","Hello","Hi")
    print redis_linsertBefore(c,"mylist","OH","Mmm")  
     # to use 'redis_lrange' for to show the list

{title="Output",lang=text,linenos=off}
    1
    2
    3
    -1
     
### linsertAfter {#linsertafter}
_**Description**_: Inserts value in a list key after the reference value pivot

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: pivot  
*string*: value  

_**Return value**_   
*number*: the length of the list after the insert, or `-1` when the value pivot was not found.

{title="Example: Using linsertAfter",lang=text,linenos=off}
    redis_del(c,"mylist")
    print redis_rpush(c,"mylist","Hello")
    print redis_rpush(c,"mylist","World")
    redis_linsertAfter(c,"mylist","Hello","--") # Returns 3
    redis_linsertAfter(c,"mylist","World","OK") # Retuns 4 
     # to use 'redis_lrange' for to show the list

### rpop {#rpop}
_**Description**_: Return and remove the last element of the list.

_**Parameters**_    
*number*: connection  
*string*: key name  

_**Return value**_   
*string*: the value, `null string` in case of empty list or no exists

{title="Example: Using rpop",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"mylist")
      C[1]="push1";C[2]="push2";C[3]="push3"
      C[4]="push4";C[5]="push5";C[6]="pushs6"
      print redis_rpush(c,"mylist",C)
      redis_lrange(c,"mylist",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      print "RPOP to empty the list 'mylist'"
      while(redis_exists(c,"mylist")) {
        print redis_rpop(c,"mylist")
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    6
    1) push1
    2) push2
    3) push3
    4) push4
    5) push5
    6) pushs6
    RPOP to empty the list 'mylist'
    pushs6
    push5
    push4
    push3
    push2
    push1

### rpoplpush {#rpoplpush}
_**Description**_: Atomically returns and removes the last element (tail) of a source list, and pushes the element at the first element (head) of a destination list.

_**Parameters**_    
*number*: connection  
*string*: the source list name  
*string*: the destination list name  

_**Return value**_   
*string*: the element being popped and pushed. If source key does not exist, `null string` is returned and no operation is performed. `-1` on error (if any of the key names exist and is not a list).

{title="Example: Using rpoplpush",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"mylist")
      C[1]="a";C[2]="b";C[3]="c";C[4]="d"
      print redis_rpush(c,"mylist",C)
       # mylist before rpoplpush is executed
      redis_lrange(c,"mylist",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      redis_del(c,"mylist0")
      print redis_rpoplpush(c,"mylist","mylist0")
      delete AR
       # mylist after rpoplpush is executed
      redis_lrange(c,"mylist",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      print "Elements in 'mylist0':"
      delete AR
      redis_lrange(c,"mylist0",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    4
    1) a
    2) b
    3) c
    4) d
    d
    1) a
    2) b
    3) c
    Elements in 'mylist0':
    1) d

### brpoplpush {#brpoplpush}
_**Description**_: Is the blocking variant of RPOPLPUSH. When the source list contains elements, this function behaves exactly like RPOPLPUSH, if the source list is empty, Redis will block the connection until another client pushes to it or until timeout is reached.

_**Parameters**_    
*number*: connection  
*string*: the source list name  
*string*: the destination list name  
*number*: timeout

_**Return value**_   
*string*: the element being popped and pushed. If timeout is reached, a `null string` is returned. `-1` on error (if any of the key names exist and is not a list).

{title="Example: Using brpoplpush",lang=text,linenos=off}
    print redis_brpoplpush(c,"mylist","mylist0",10)

### lpop {#lpop}
_**Description**_: Return and remove the first element of the list.

_**Parameters**_    
*number*: connection  
*string* key name  

_**Return value**_   
*string*: the value, `null string` in case of empty list or no exists

{title="Example: Using lpop",lang=text,linenos=off}
    @load "redis"
    BEGIN{
     c=redis_connect()
     ret=redis_del(c,"list1")
     print "return del="ret
     ret=redis_lpush(c,"list1","AA")
     print "return lpush="ret
     ret=redis_lpush(c,"list1","BB")
     print "return lpush="ret
     ret=redis_lpush(c,"list1","CC")
     print "return lpush="ret
     ret=redis_lrange(c,"list1",AR,0,-1)
     print "return lrange="ret
     for(i in AR) {
       print i": "AR[i]
     }
     ret=redis_lpop(c,"list1")
     print "return lpop="ret
     delete AR
     ret=redis_lrange(c,"list1",AR,0,-1)
     print "return lrange="ret
     for(i in AR) {
       print i": "AR[i]
     }
     redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    return del=1
    return lpush=1
    return lpush=2
    return lpush=3
    return lrange=1
    1: CC
    2: BB
    3: AA
    return lpop=CC
    return lrange=1
    1: BB
    2: AA

### lpush {#lpush}
_**Description**_: Adds all the specified values to the head (left) of the list. Creates the list if the key didn't exist.

_**Parameters**_    
*number*: connection  
*key*: key name  
*string or array*: the string value to push in key, or if is an array, it's containing all values.

_**Return value**_   
*number*: The new length of the list in case of success, `-1` on error (if the key exists and is not a list).

{title="Example: Using lpush",lang=text,linenos=off}
    redis_lpush(c,"list1","dd")
    # being the array 'A' that containing the values
    redis_lpush(c,"list1",A) 
    # to see example code of rpush function

### lpushx {#lpushx}
_**Description**_: Inserts a value at the head of the list, only if the key already exists and holds a list, no operation will be performed when key does not yet exist.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: the value to push in key  

_**Return value**_   
*number*: The new length of the list in case of success. `0` when no operation is executed. `-1` on error (if the key exists and is not a list).

{title="Example: Using lpushx",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"mylist99")
      redis_lpush(c,"mylist99","s1")
      redis_lpushx(c,"mylist99","s2") # returns 2
      redis_del(c,"mylist99")
       # The next returns 0. The list not exist
      redis_lpushx(c,"mylist99","a")
      redis_lpush(c,"mylist99","a")  # returns 1
      redis_close(c)
    }

### rpushx {#rpushx}
_**Description**_: Inserts a value at the tail of the list, only if the key already exists and holds a list, no operation will be performed when key does not yet exist.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string*: the value to push in key  

_**Return value**_   
*number*: The new length of the list in case of success. `0` when no operation is executed. `-1` on error (if the key exists and is not a list).

{title="Example: Using rpushx",lang=text,linenos=off}
    redis_del(c,"mylist99")
     # the next returns 0 because 'mylist99' not exist
    redis_rpushx(c,"mylist99","ppp") 
    redis_rpush(c,"mylist99","s0")
    redis_lpush(c,"mylist99","s1")
    redis_rpushx(c,"mylist99","s2")  # returns 3

### rpush {#rpush}
_**Description**_: Adds all the specified values to the tail (right) of the list. Creates the list if the key didn't exist.

_**Parameters**_    
*number*: connection  
*string*: key name  
*string or array*: the string value to push in key, or if is an array, it's containing all values.

_**Return value**_   
*number*: The new length of the list in case of success, `-1` on error (if the key exists and is not a list).

{title="Example 1: Using rpush",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"mylist")
      C[1]="Hello";C[2]="World"
      print redis_rpush(c,"mylist",C) 
      redis_lrange(c,"mylist",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      C[1]="push1";C[2]="push2"
      print redis_lpush(c,"mylist",C)
      delete AR
      redis_lrange(c,"mylist",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    2
    1) Hello
    2) World
    4
    1) push2
    2) push1
    3) Hello
    4) World

{title="Example 2: Using rpush",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"mylist")
      r=redis_rpush(c,"mylist","Hello")
      print r
      r=redis_rpush(c,"mylist","World")
      print r
      redis_lrange(c,"mylist",AR,0,-1)
      for(i in AR) {
        print i") "AR[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1
    2
    1) Hello
    2) World

### lrange {#lrange}
_**Description**_: Returns the specified elements of the list stored at the specified key in the range [start, end]. start and stop are interpreted as indices:
0 the first element, 1 the second ...
-1 the last element, -2 the penultimate ...

_**Parameters**_    
*number*: connection  
*string*: key name  
*array*: for the result. It will contain the values in specified range  
*number*: start   
*number*: end  

_**Return value**_   
`1` on success, `0` in case of empty list or no exists

{title="Example: Using lrange",lang=text,linenos=off}
    # it range includes all values.
    redis_lrange(c,"list1",AR,0,-1)

### lrem {#lrem}
_**Description**_: Removes the first count occurences of the value element from the list. If count is zero, all the matching elements are removed. If count is negative, elements are removed from tail to head.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: count  
*string*: value   

_**Return value**_   
*number*: the number of removed elements, `-1` on error

{title="Example: Using lrem",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_del(c,"list1")
      redis_lpush(c,"list1","AA")
      redis_lpush(c,"list1","BB")
      redis_lpush(c,"list1","CC")
      redis_lpush(c,"list1","BB")
      redis_lrange(c,"list1",AR,0,-1)
      for(i in AR) {
        print i": "AR[i]
      }
       # count is 4 but removes only two (existing values)
      ret=redis_lrem(c,"list1",4,"BB") 
      print "return redis_lrem="ret
      if(ret==-1) print ERRNO
      delete AR
      ret=redis_lrange(c,"list1",AR,0,-1)
      for(i in AR) {
         print i": "AR[i]
      }
      redis_close(c)
    }

### lset {#lset}
_**Description**_: Set the list at index with the new value.

_**Parameters**_    
*number*: connection  
*string*: key name   
*number*: index  
*string*: value  

_**Return value**_   
`1` if the new value is setted. `-1` on error (if the index is out of range, or data type identified by key is not a list).

{title="Example: Using lset",lang=text,linenos=off}
    @load "redis"
    BEGIN{
     c=redis_connect()
     # set "28" in list2 with index 7
     ret=redis_lset(c,"list2",7,"28")
     print "lset returns "ret
     redis_close(c)
    }

### ltrim {#ltrim}
_**Description**_: Trims an existing list so that it will contain only a specified range of elements.
It is recommended that you consult on [possibles uses](http://redis.io/commands/ltrim) of this function in the main page of Redis project.

_**Parameters**_    
*number*: connection  
*string*: key name  
*number*: start  
*number*: stop  

_**Return value**_   
`1` on success, `-1` on error (by example a WRONGTYPE Operation). Out of range indexes will not produce an error.

{title="Example: Using ltrim",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      ret=redis_lrange(c,"list2",AR,0,-1)
      if(ret==1) {
        print "--->Values in list2<---"
        for(i in AR) {
          print i": "AR[i]
        }
        ret=redis_ltrim(c,"list2",2,5)
        if(ret==1) {
          delete AR
          redis_lrange(c,"list2",AR,0,-1)
          print "--->Values in list2 after applying",
	  print "'ltrim'<---"
          for(i in AR) {
            print i": "AR[i]
          }
        }
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    --->Values in list2<---
    1: 96
    2: 5
    3: 63
    4: 60
    5: 12
    6: 69
    7: 162
    --->Values in list2 after applying 'ltrim'<---
    1: 63
    2: 60
    3: 12
    4: 69

### brpop {#brpop}
_**Description**_: Is a blocking list pop primitive. Pops elements from the tail of a list.
To see [Redis site](http://redis.io/commands/brpop) for a more detailed explanation

_**Parameters**_    
*number*: connection  
*string or array*: key name (the list name), or an array containing the list names  
*array*: for the results. If a elemment be popped then, this array is two-element where the first element is the name of the key where it value was popped and the second element is the value of the popped element  
*number*: timeout  

_**Return value**_   
`1`, if popped a element. A `string null` when no element could be popped and the timeout expired.

{title="Example: Using brpop",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      redis_del(c,"listb")
      redis_lpush(c,"listb","hello")
      redis_lpush(c,"listb","Sussan")
      redis_lpush(c,"listb","nice")
      LIST[1]="listbbb"; LIST[2]="listbb"; LIST[3]="listb"
       # knowing that "listbbb" and "listbb" does not exist
       # brpop will get a element from 'listb'
      redis_brpop(c,LIST,AR,10) # return is 1
      for(i in AR) {
        print i": "AR[i]
      }
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    1: listb
    2: hello

### blpop {#blpop}
_**Description**_: Is a blocking list pop primitive. Pops elements from the head of a list.
To see [Redis site](http://redis.io/commands/blpop) for a more detailed explanation

_**Parameters**_    
*number*: connection  
*string or array*: key name (the list name), or an array containing the list names  
*array*: for the results. If a elemment be popped then, this array is two-element where the first element is the name of the key where it value was popped and the second element is the value of the popped element.   
*number*: timeout  

_**Return value**_   
`1`, if popped a element. A `string null` when no element could be popped and the timeout expired.

{title="Example: Using blpop",lang=text,linenos=off}
    # the same example code in brpop
    # ...
    redis_blpop(c,LIST,AR,10) # returns is 1

{title="Output",lang=text,linenos=off}
    1: listb
    2: nice

### llen {#llen}
_**Description**_: Returns the size of a list identified by Key.

If the list didn't exist or is empty, the command returns 0. If the data type identified by Key is not a list, the command returns `-1`.

_**Parameters**_    
*number*: connection  
*string*: key name  

_**Return value**_   
*number*: the size of the list identified by key, `0` if the key no exist or is empty, `-1` on error (if the data type identified by key is not list)

{title="Example: Using llen",lang=text,linenos=off}
    print "Length of 'mylist': "redis_llen(c,"mylist")


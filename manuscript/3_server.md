# Server {#server}

* [dbsize](#dbsize) - Returns the number of keys in the currently-selected database
* [flushdb](#flushdb) - Deletes all the keys of the currently selected DB
* [info](#info) - Returns information and statistics about the server

### dbsize {#dbsize}
_**Description**_: Returns the number of keys in the currently-selected database

_**Parameters**_    
*number*: connection handle  

_**Return value**_    
*number*: `the number of keys` in the DB   

{title="Example: Using dbsize",lang=text,linenos=off}
    @load "redis"
    BEGIN{
      c=redis_connect()
      redis_select(c,5) # DB 5 selected
      print "DBSIZE:",
            redis_dbsize(c) # number of keys into DB 5
      print "FLUSHDB:",
            redis_flushdb(c) # delete all the keys of DB 5
      print redis_keys(c,"*",AR)
      print "DBSIZE: "redis_dbsize(c)
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    DBSIZE: 3
    FLUSHDB: 1
    0
    DBSIZE: 0

### flushdb {#flushdb}
_**Description**_: Delete all the keys of the currently selected DB

_**Parameters**_    
*number*: connection handle  

_**Return value**_    
`1` on success  

{title="Example: Using flushdb",lang=text,linenos=off}
    c=redis_connect()
    redis_flushdb(c) 
     # deletes all the keys of the currently DB

### info {#info}
_**Description**_: Returns information and statistics about the server.
If is executed as pipelined command, the return is an string; this string is an collection of text lines. Lines can contain a section name (starting with a # character) or a property. All the properties are in the form of field:value terminated by \r\n   

_**Parameters**_    
*number*: connection handle   
*array*: is an associative array and stores the results   
*string*: is optional, and admits a name of section to filter out the scope of this section  

_**Return value**_    
`1` on success, `-1` on error  

{title="Example 1: Using info",lang=text,linenos=off}
    @load "redis"
    BEGIN{
     c=redis_connect()
     r=redis_info(c,AR)
     for(i in AR) {
       print i" ==> "AR[i]
     }
     redis_close(c)
    }

{title="Example 2: Using info with pipelining",lang=text,linenos=off}
    @load "redis"
    BEGIN {
     c=redis_connect()
     p=redis_pipeline(c)
     redis_info(p,AR,"replication") # asks a section
     # here others commands to pipeline
     #
     string_result=redis_getReply(p,ARRAY)
      # string_result contains the result as an
      # string multiline
     n=split(string_result,ARRAY,"\r\n")
     for(i in ARRAY) {
       n=split(ARRAY[i],INFO,":")
       if(n==2) {
         print INFO[1]" ==> "INFO[2]
       }
     }
     redis_close(c)
    }


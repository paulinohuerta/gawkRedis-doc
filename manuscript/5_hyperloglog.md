# HyperLogLog Functions {#hyperloglog}
Recommended reading [Redis HyperLogLog](http://redis.io/commands/pfadd)

* [pfadd](#pfadd) - Adds elements to the HyperLogLog data structure.
* [pfcount](#pfcount) - Returns the approximated cardinality computed by the HyperLogLog data structure stored at the specified key.
* [pfmerge](#pfmerge) - Merge multiple HyperLogLog keys into an unique key.

### pfadd {#pfadd}
_**Description**_: Adds elements to the HyperLogLog data structure stored at the key specified.

_**Parameters**_   
*number*: connection    
*string*: key name    
*string or array*: a element or an array containing the elements

_**Return value**_    
*number*: `1` if at least 1 HyperLogLog internal register was altered. `0` otherwise.

{title="Example: Using pfadd",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      AR[1]="a"; AR[2]="b"; AR[3]="c"
      AR[4]="d"; AR[5]="e"; AR[6]="f"
      redis_pfadd(c,"hll",AR) # returns 1
      redis_pfcount(c,"hll")  # returns 6
      redis_close(c)
    }

### pfcount {#pfcount}
_**Description**_: Returns the approximated cardinality computed by the HyperLogLog data structure stored at the specified key.

_**Parameters**_   
*number*: connection    
*string or array*: a key name or an array containing the key names    

_**Return value**_    
*number*: The approximated number of unique elements observed via PFADD. `0` if the key does not exist.

{title="Example: Using pfcount",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      AR[1]="foo"
      AR[2]="bar"
      AR[3]="zap"
      redis_pfadd(c,"hll",AR) # returns 1
      AR[1]=AR[2]="zap"
      redis_pfadd(c,"hll",AR) # returns 0
      BR[1]="foo"
      BR[2]="bar"
      redis_pfadd(c,"hll",BR) # returns 0
      print redis_pfcount(c,"hll")
      #
      CR[1]=1; CR[2]=2; CR[3]=3
      redis_pfadd(c,"other-hll",CR) # returns 1
      K[1]="hll"
      K[2]="other-hll"
      print redis_pfcount(c,K)
      redis_close(c)
    }

{title="Output",lang=text,linenos=off}
    3
    6

### pfmerge {#pfmerge}
_**Description**_: Merge multiple HyperLogLog keys into an unique key that will approximate the cardinality of the union of the observed Sets of the source HyperLogLog structures.

_**Parameters**_    
*number*: connection    
*string*: a destination key name    
*string or array*: a source key name or an array containing the source key names

_**Return value**_    
*number*: returns `1`.

{title="Example: Using pfmerge",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      AR[1]="foo"; AR[2]="bar"; AR[3]="zap"; AR[4]="a"
      redis_pfadd(c,"hll1",AR) # returns 1
      BR[1]="a"; BR[2]="b"; BR[3]="c"; BR[4]="foo"
      redis_pfadd(c,"hll2",BR) # returns 1
      K[1]="hll1"; K[2]="hll2"
      redis_pfmerge(c,"hll3",K) # returns 1
      redis_pfcount(c,"hll3") # returns 6
      redis_close(c)
    }

# Geolocation Functions {#geolocation}
Recommended reading [Redis Geolocation](http://redis.io/commands/geoadd).   
Geospatial data (latitude, longitude, name) are stored into a key as a sorted set, in a way that makes it possible to later retrieve items using a query by radius or member.

1. [geoadd](#geoadd) - Adds the specified geospatial items to one specified key.
1. [geodist](#geodist) - Obtains the distance between two members with information geospatial.
1. [geohash](#geohash) - Returns members of a geospatial index as standard geohash strings.
1. [geopos](#geopos) - Returns longitude and latitude of members of a geospatial index.
1. [georadius](#georadius) - Obtains the members with geospatial information which are within the borders of the area specified with the center and the maximum distance from the center.
1. [georadiusWD](#georadiuswd) - This is like `georadius`, adding `distance` to the results.
1. [georadiusWC](#georadiuswc) - This is like `georadius`, adding coordinates (longitude and latitude) to the results.
1. [georadiusWDWC](#georadiuswdwc) - This is like `georadius`, adding distance and coordinates to the results.
1. [georadiusbymember](#georadiusbymember) - This is like `georadius` with the same results. It takes the name of a member existing in a geospatial index
1. [georadiusbymemberWD](#georadiusbymemberwd) - This is like `georadiusbymember`, adding `distance` to the results.
1. [georadiusbymemberWC](#georadiusbymemberwc) - This is like `georadiusbymember`, adding coordinates (longitude and latitude) to the results.
1. [georadiusbymemberWDWC](#georadiusbymemberwdwc) - This is like `georadiusbymember`, adding distance and coordinates to the results.


### geoadd {#geoadd}
_**Description**_: Adds the specified geospatial items (latitude, longitude, name) to the specified key. 

_**Parameters**_
*number*: connection  
*string*: key name  
*array*:  it contains three elements (longitude, latitude, name) per item

_**Return value**_
*number*: the number of elements added to the sorted set, not including elements already existing for which the score was updated.

{title="Example: Using geoadd",lang=awk,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      A[1]="-118.2436800"
      A[2]="34.0522300"
      A[3]="la"
      A[4]="-74.0059700"
      A[5]="40.7142700"
      A[6]="nyc"
      redis_geoadd(c,"US",A) # returns 2, are two items added to a zset
      print "la-nyc kms: "redis_geodist(c,"US","la","nyc","km")
      print "la-nyc miles: "redis_geodist(c,"US","la","nyc","mi")
      redis_close(c)
    }

Output:

    la-nyc kms: 3936.8457102104558
    la-nyc miles: 2446.248592721523

### geodist {#geodist}
_**Description**_: Returns the distance between two members in the geospatial index represented by the sorted set.

_**Parameters**_
*number*: connection  
*string*: key name  
*string*: name member  
*string*: name member  
*optional string*: the unit, must be one of the following values, m, km, mi, ft. Defaults to meters.  

_**Return value**_
*number*: represented as a string in the specified unit, or null string if one or both the members are missing

{title="Example: Using geodist",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      print redis_geodist(c,"US","la","nyc","m")
      redis_close(c)
    }

Output:

    3936845.7102104556

### georadius {#georadius}
_**Description**_: Returns the members of a sorted set populated with geospatial information using `geoadd`, which are within the borders of the area specified with the center location and the maximum distance from the center (the radius).

_**Parameters**_
*number*: connection  
*string*: key name  
*array*: will contain the results, a set of strings.  
*number*: longitud  
*number*: latitud  
*number*: radius  
*string*: with a value between m|km|ft|mi
*optional string*: the order with values desc or asc   
*optional number*: to limit the results to the first N matching items. N is passed to `count` option of the command.

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadius",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      A[1]="13.361389"
      A[2]="38.115556"
      A[3]="Palermo"
      A[4]="15.087269"
      A[5]="37.502669"
      A[6]="Catania"
      A[7]="12.5372"
      A[8]="38.0176"
      A[9]="Trapani"
      c=redis_connect()
      redis_geoadd(c,"sicilia",A)
      print redis_geodist(c,"sicilia","Catania","Trapani","km")
      redis_georadius(c,"sicilia",AR,15,37,200,"km")
        # georadius using optionals argumments
        # redis_georadius(c,"sicilia",AR,15,37,200,"km",1)  # using count
        # redis_georadius(c,"sicilia",AR,15,37,200,"km","desc", 1)  # order and count
        # redis_georadius(c,"sicilia",AR,15,37,200,"km","desc")  # only order
      dumparray(AR,"NN") # function defined in the geopos example
      redis_close(c)
    }

Output:

    231.42622077769485
    1) Palermo
    2) Catania

### geohash {#geohash}
_**Description**_: Returns members of a geospatial index as standard geohash strings.

_**Parameters**_
*number*: connection  
*string*: key name  
*array*: it contains the names of members   
*array*: will contain the results. Each element is the Geohash corresponding to each member name passed as argument

_**Return value**_
`1` on success. `0` if not exists the key. `-1` on error.

{title="Example: Using geohash",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      A[1]="Trapani"
      A[2]="Catania"
      c=redis_connect()
      redis_geohash(c,"sicilia",A,RESP)
      for(i=1; i<=2; i++) {
        print i") "RESP[i]
      }
      redis_close(c)
    }

Output:

    1) sqbbm2ck9f0
    2) sqdtr74hyu0

### geopos {#geopos}
_**Description**_: Returns longitude and latitude of members of a geospatial index.

_**Parameters**_
*number*: connection   
*string*: key name   
*array*: it contains the names of members   
*array*: will contain the results where each element is a two elements array representing longitude and latitude (x,y) of each member name passed as argument.  Non existing elements are reported as NULL elements of the array.

_**Return value**_
`1` on success. `0` if not exists the key. `-1` on error.

{title="Example: Using geopos",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect() 
      B[1]="Trapani"; B[2]="Catanzaro"; B[3]="Catania"
      redis_geopos(c,"sicilia",B,AR) # returns 1
      redis_close(c)  
      if(length(AR)>0) {
         dumparray(AR,"NN")
      }
    }

    function dumparray(array,e, i) {
      for (i in array){
        if (isarray(array[i])){
          dumparray(array[i],e "[\"" i "\"]")
        }
        else {
            printf("%s[\"%s\"] = %s\n",e,i, array[i])
        }
      }
    }

Output:

    NN["1"]["1"] = 12.537200152873993
    NN["1"]["2"] = 38.017599561572482
    NN["3"]["1"] = 15.087267458438873
    NN["3"]["2"] = 37.502668423331613

## georadiusWD {#georadiuswd}
_**Description**_: This is like `georadius`, adding `distance` to the results.

_**Parameters**_
*number*: connection   
*string*: key name   
*array*: will contain the results, a set of strings.   
*number*: longitud   
*number*: latitud   
*number*: radius   
*string*: with a value between m|km|ft|mi
*optional string*: order 
*optional number*: count  

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusWD",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      redis_zrange(c,"sisu",RET,0,-1) # returns 1
      print "zset sisu contains index geospatial for"
      for(i in RET) {
        print i": "RET[i]
      }
      redis_georadius(c,"sisu",AR,"14",37.9,150,"km",1)
      for(i in AR) {
        print i") "AR[i]
      }
      print "georadiusWD radius 2500 output in km desc order "
      delete(AR)
      redis_georadiusWD(c,"sisu",AR,"14",37.9,2500,"km","desc") # returns 1
      dumparray(AR,"NN") # function defined in the geopos example
      redis_close(c)
    }

## georadiusWC {#georadiuswc}
_**Description**_: This is like `georadius`, adding `coordinates` to the results.

_**Parameters**_
*number*: connection    
*string*: key name    
*array*: will contain the results, a set of strings.   
*number*: longitud   
*number*: latitud   
*number*: radius    
*string*: with a value between m|km|ft|mi
*optional string*: order 
*optional number*: count 

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusWC",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      print "georadiusWC radius 2500 output in km desc order "
      delete(AR)
      redis_georadiusWC(c,"sisu",AR,"14",37.9,2500,"km","desc") # returns 1
      dumparray(AR,"NN")  # function defined in the geopos example
      redis_close(c)
    }

## georadiusWDWC {#georadiuswdwc}
_**Description**_: This is like `georadius`, adding `distance` and `coordinates` to the results.

_**Parameters**_
*number*: connection   
*string*: key name  
*array*: will contain the results, a set of strings.   
*number*: longitud    
*number*: latitud   
*number*: radius   
*string*: with a value between m|km|ft|mi  
*optional string*: order 
*optional number*: count  

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusWDWC",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect()
      print "georadiusWDWC radius 2500 output in km desc order "
      delete(AR)
      redis_georadiusWDWC(c,"sisu",AR,"14",37.9,2500,"km","desc") # returns 1
      dumparray(AR,"NN")  # function defined in the geopos example
      redis_close(c)
    }

## georadiusbymember {#georadiusbymember}
_**Description**_: This command is exactly like `georadius`. The difference is that instead of to take a longitude and latitude as the center of the area, it takes the name of a member already existing inside the geospatial index.

_**Parameters**_
*number*: connection  
*string*: key name   
*array*: will contain the results, a set of strings.   
*string*: name member   
*number*: radius   
*string*: with a value between m|km|ft|mi

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusbymember",lang=text,linenos=off}
    @load "redis"
    BEGIN {
      c=redis_connect() 
      redis_zrangeWithScores(c,"sisu",RES,0,-1) # returns 1
      for(i in RES){
        print i") "RES[i]
      }
      print ""
      redis_georadiusbymember(c,"sisu",AR,"Ecija",350,"km") # returns 1
      dumparray(AR,"NN")
      delete AR
      print ""
      redis_georadiusbymember(c,"sisu",AR,"Ecija",2800,"km") # returns 1
      redis_close(c)
      dumparray(AR,"NN")
    }

Output:

    1) Sevilla
    2) 1966655518805908
    3) Ecija
    4) 1968142286694693
    5) Palermo
    6) 3479099956230698
    7) Catania
    8) 3479447370796909

    NN["1"] = Sevilla
    NN["2"] = Ecija

    NN["1"] = Sevilla
    NN["2"] = Ecija
    NN["3"] = Palermo
    NN["4"] = Catania
    
## georadiusbymemberWD {#georadiusmemberwd}
_**Description**_: Returns the members of a sorted set populated with geospatial information using `geoadd`, adding `distance` to the results.

_**Parameters**_
*number*: connection  
*string*: key name  
*array*: will contain the results, a set of strings.   
*string*: name member   
*number*: radius  
*string*: with a value between m|km|ft|mi   

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusbymemberWD",lang=text,linenos=off}
    @load "redis"
    BEGIN {
     c=redis_connect() 
     redis_georadiusbymemberWD(c,"sisu",AR,"Ecija",2800,"km")
     redis_close(c)
     dumparray(AR,"NN")
    }

Output:

    NN["1"]["1"] = Sevilla
    NN["1"]["2"] = 81.3977
    NN["2"]["1"] = Ecija
    NN["2"]["2"] = 0.0000
    NN["3"]["1"] = Palermo
    NN["3"]["2"] = 1618.9443
    NN["4"]["1"] = Catania
    NN["4"]["2"] = 1775.8787

## georadiusbymemberWC {#georadiusbymemberwc}
_**Description**_: Returns the members of a sorted set populated with geospatial information using `geoadd`, adding `coordinates` to the results.

_**Parameters**_
*number*: connection   
*string*: key name   
*array*: will contain the results, a set of strings.  
*string*: name member   
*number*: radius  
*string*: with a value between m|km|ft|mi  

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusbymemberWC",lang=text,linenos=off}
    @load "redis"
    BEGIN {
     c=redis_connect()
     redis_georadiusbymemberWC(c,"sisu",AR,"Ecija",2800,"km")
     redis_close(c)
     dumparray(AR,"NN")
    }

Output:

    NN["1"]["1"] = Sevilla
    NN["1"]["2"]["1"] = -5.9844991564750671
    NN["1"]["2"]["2"] = 37.389100241787432
    NN["2"]["1"] = Ecija
    NN["2"]["2"]["1"] = -5.0826975703239441
    NN["2"]["2"]["2"] = 37.541500351492687
    NN["3"]["1"] = Palermo
    NN["3"]["2"]["1"] = 13.361389338970184
    NN["3"]["2"]["2"] = 38.115556395496299
    NN["4"]["1"] = Catania
    NN["4"]["2"]["1"] = 15.087267458438873
    NN["4"]["2"]["2"] = 37.502668423331613
       
## georadiusbymemberWDWC {#georadiusbymemberwdwc}
_**Description**_: Returns the members of a sorted set populated with geospatial information using `geoadd`, adding `distances` and `coordinates` to the results.

_**Parameters**_
*number*: connection  
*string*: key name  
*array*: will contain the results, a set of strings.   
*string*: name member   
*number*: radius   
*string*: with a value between m|km|ft|mi   

_**Return value**_
`1` if is at least one result. `0` if there is no result. `-1` on error.

{title="Example: Using georadiusbymemberWDWC",lang=text,linenos=off}
    @load "redis"
    BEGIN {
     c=redis_connect() 
     redis_georadiusbymemberWDWC(c,"sisu",AR,"Ecija",2800,"km")
     redis_close(c)
     dumparray(AR,"NN")
    }

Output:

    NN["1"]["1"] = Sevilla
    NN["1"]["2"] = 81.3977
    NN["1"]["3"]["1"] = -5.9844991564750671
    NN["1"]["3"]["2"] = 37.389100241787432
    NN["2"]["1"] = Ecija
    NN["2"]["2"] = 0.0000
    NN["2"]["3"]["1"] = -5.0826975703239441
    NN["2"]["3"]["2"] = 37.541500351492687
    NN["3"]["1"] = Palermo
    NN["3"]["2"] = 1618.9443
    NN["3"]["3"]["1"] = 13.361389338970184
    NN["3"]["3"]["2"] = 38.115556395496299
    NN["4"]["1"] = Catania
    NN["4"]["2"] = 1775.8787
    NN["4"]["3"]["1"] = 15.087267458438873
    NN["4"]["3"]["2"] = 37.502668423331613


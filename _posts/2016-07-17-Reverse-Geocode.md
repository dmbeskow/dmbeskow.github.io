---
layout: post
title:  "Three Methods for Reverse Geocode"
categories: [R]
tags: [spatial]
---

In summary, this post will demonstrate three methods for reverse geocoding.  These three methods are summarized below:

1. Extract country and state/province data from shapefile
2. Identify closest city 
3. Identify closest zip code (US Only)

In my years of munging geospatial data, I've run into a number of instances when I have a data set with a lat-lon combination, but no other spatial reference (i.e. Country, State/Province, City, etc.).  If you have a data set with latitude and longitude but not _Country_, _State/Province_, or _city_, than I'd recommend using one of the methods below (or a combination of them) to add a _Country_, _State/Province_, and/or _City_ field to your data.  You can think of this as a type of reverse geocode.  If you have US Only spatial data, I'd recommend using the zip code method below.  If you have international data, and require _Country_ and _State/Province_, use the shapefile data extraction method below.  If you require _Country_ and _City_ data, use the world city data in the second method below.  If you require _Country_, _State/Province_, and _City_ data, you will have to use a combination of both the shapefile data extraction and international city data methods.  

## Extract Data from Shapefile

This method will extract spatial data from a shapefile.  This is most often conducted to enrich your original data with _Country_ and _State/Province_ data.  This method is the most accurate method for adding _country_ and _state/province_ since it uses point-in-polygon calculations.  The shapefile data is also usually more accurate than the city data used in the next example.

To manipulate shapefiles, we first need to load the _sp_ and _maptools_ packages.


{% highlight r %}
library(sp)
library(maptools)
{% endhighlight %}
Next you will need to download a Level 1 World Shapefile.  I've found a decent shapefile that Harvard maintains at the following link: [Harvard Level 1 World Shapefile](http://worldmap.harvard.edu/data/geonode:g2008_1).  

After downloading and unzipping the shapefile, we read it into our environment below:

{% highlight r %}
world.shp <- readShapePoly("g2008_1.shp")
proj4string(world.shp) <- CRS("+proj=longlat +ellps=WGS84")
{% endhighlight %}



This is what our Level 1 shapefile looks like (remember that Level 1 Shapefiles are one level below country boundary for all countries).



![alt text](world2.png)



Now I'm going to read in a list of lat/lon combinations from around the world to test our code on.  While these aren't explicitly named, note that they were created by geocoding the following cities: Sydney, London, Moscow, Beijing, Oklahoma City, Nairobi, Berlin, and Rio de Janero.


{% highlight r %}
## Choose random cities to test our algorithms
cities <- data.frame(
  lat=c(-33.86785 , 51.50726 , 55.75000 , 39.90750 , 35.46756 , -1.28333 , 52.52330 ,-22.90278),
  lon=c(151.20732 , -0.1278328 , 37.5 ,116.39723 ,-97.51643  ,36.8166700 , 13.41377, -43.2075)
)
{% endhighlight %}

Now let's see a picture of our cities plotted over our shapefile:



![points on shapefile](world.png)

Next, we need to convert the cities data frame to a Spatial Points Data Frame.  We start by creating a separate object called _cities.sp_, and then creating the Spatial Points Data Frame.


{% highlight r %}
cities.sp <- cities                              ## Rename
coordinates(cities.sp) <- ~lon+lat               ## Create Spatial Points Data Frame
proj4string(cities.sp) <- CRS("+proj=longlat +ellps=WGS84")
{% endhighlight %}

Next we will use point-in-polygon calculations to determine which polygon each point fall in.  The _over_ command in R is a powerful command that allows us to conduct this point-in-polygon calculation.  


{% highlight r %}
## Conduct point-in-polygon calculation and extract data
extracted.data <- over(cities.sp,world.shp)

extracted.data
{% endhighlight %}



{% highlight text %}
##          AREA PERIMETER G2008_1_ G2008_1_ID ADM1_CODE ADM0_CODE
## 1 76.60676282 78.223011    32966      32965       470        17
## 2 17.22851535 50.129004     9630       9629      3182       256
## 3  0.14140157  1.653553     9584       9583      2535       204
## 4  1.73808134  7.161621    13166      13165       899        53
## 5 18.00501971 25.756017    14010      14009      3250       259
## 6  0.05658006  1.664051    27161      27160     51328       133
## 7  0.11789101  2.070110    10602      10601      1310        93
## 8  3.83091826 21.150444    32532      32531       683        37
##                                    ADM0_NAME       ADM1_NAME
## 1                                  Australia New South Wales
## 2 U.K. of Great Britain and Northern Ireland         England
## 3                         Russian Federation          Moskva
## 4                                      China     Beijing Shi
## 5                   United States of America        Oklahoma
## 6                                      Kenya         Nairobi
## 7                                    Germany          Berlin
## 8                                     Brazil  Rio De Janeiro
##   LAST_UPDAT CONTINENT                    REGION STR_YEAR0 STR_YEAR1
## 1   20050415   Oceania Australia and New Zealand         0         0
## 2   20050825    Europe           Northern Europe         0         0
## 3   20050415    Europe            Eastern Europe         0         0
## 4   20050415      Asia              Eastern Asia         0         0
## 5   20060103  Americas          Northern America         0         0
## 6   20081111    Africa            Eastern Africa         0         0
## 7   20051229    Europe            Western Europe         0         0
## 8   20050415  Americas             South America         0         0
##   EXP_YEAR0 EXP_YEAR1
## 1         0         0
## 2         0         0
## 3         0         0
## 4         0         0
## 5         0         0
## 6         0         0
## 7         0         0
## 8         0         0
{% endhighlight %}


Next we will merge this data with our original data:


{% highlight r %}
## Merge Data
cities$Country <- extracted.data$ADM0_NAME
cities$level_1 <- extracted.data$ADM1_NAME
{% endhighlight %}

Below is our final data frame.

<!-- html table generated in R 3.2.1 by xtable 1.8-2 package -->
<!-- Sun Jul 24 23:06:13 2016 -->
<table border=1>
<tr> <th>  </th> <th> lat </th> <th> lon </th> <th> Country </th> <th> level_1 </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> -33.87 </td> <td align="right"> 151.21 </td> <td> Australia </td> <td> New South Wales </td> </tr>
  <tr> <td align="right"> 2 </td> <td align="right"> 51.51 </td> <td align="right"> -0.13 </td> <td> U.K. of Great Britain and Northern Ireland </td> <td> England </td> </tr>
  <tr> <td align="right"> 3 </td> <td align="right"> 55.75 </td> <td align="right"> 37.50 </td> <td> Russian Federation </td> <td> Moskva </td> </tr>
  <tr> <td align="right"> 4 </td> <td align="right"> 39.91 </td> <td align="right"> 116.40 </td> <td> China </td> <td> Beijing Shi </td> </tr>
  <tr> <td align="right"> 5 </td> <td align="right"> 35.47 </td> <td align="right"> -97.52 </td> <td> United States of America </td> <td> Oklahoma </td> </tr>
  <tr> <td align="right"> 6 </td> <td align="right"> -1.28 </td> <td align="right"> 36.82 </td> <td> Kenya </td> <td> Nairobi </td> </tr>
  <tr> <td align="right"> 7 </td> <td align="right"> 52.52 </td> <td align="right"> 13.41 </td> <td> Germany </td> <td> Berlin </td> </tr>
  <tr> <td align="right"> 8 </td> <td align="right"> -22.90 </td> <td align="right"> -43.21 </td> <td> Brazil </td> <td> Rio De Janeiro </td> </tr>
   </table>

Now we have spatial data added to our original data frame and we can easily bin data by _country_ or _province_.

## Reverse Geocode to City Level (with OpenGeoCode Data)



The next method I'll demonstrate is to find the nearest world city.  There are several data bases available online for doing this.  Open Geo Code maintains a database at the following [link](http://www.opengeocode.org/download.php#cities).  This database is maintaned by the National Geospatial Agency (NGA) and is fairly comprehensive.  It does contain multiple entries for each city (usually to capture various spellings of the same city).  My biggest complaint of this data set is that it does not contain any cities for the US or China.  While I can sort of understand not having any US cities, I have no idea why it does not contain Chinese cities.  

![alt text](worldcities.png)

The other data set that I've used before is from Maxmind [link](http://dev.maxmind.com/geoip/legacy/geolite/).  This data set is not maintained as much as the NGA data set, but does contain all countries (to include US cities).  This is a fairly large dataset, containing over 3 million entries.

Both data sets require some cleaning.  The NGA dataset contains a unique feature ID, which can be used to remove duplicates.  Ensure that you maintain either an english or latin spelling of city names.  The Maxmind data set does not contain a unique feature ID, so I had to create one by concatenating the lat/lon combination.  

















### With Maxmind Data

For this blog post, I will only demonstrate using the Maxmind data.  First we'll read in the data:


{% highlight r %}
world.cities <- read.csv("worldcitiespop.txt",as.is=TRUE)
{% endhighlight %}



Next we will concatenate the lat/lon combination in order to create a unique identifier.  Having done this, we remove duplicates:


{% highlight r %}
## Create identifier with lat/lon
world.cities$id<-paste(world.cities$Latitude,world.cities$Longitude,sep="")

## Remove Duplicates
world.cities <- world.cities[!duplicated(world.cities$id),]
{% endhighlight %}

Next, we will use the _fields_ package to create a distance matrix, and then choose the closest city to each of these grids.  Note that if you have grids near borders, this algorithm may find that the closest city is on the other side of the border.  


{% highlight r %}
library(fields)

## Create distance matrix
mat<-rdist.earth(data.matrix(cities[,c(2,1)]),data.matrix(world.cities[,c(7,6)]))

## Add fields to data frame
cities$country <- NA
cities$city <- NA
cities$accent_city <- NA
cities$dist <- NA

## Loop through data frame and find closes city, merging country and level 1 data 
for(i in 1:nrow(cities)){
k<-which.min(mat[i,])
cities$country[i] <- world.cities$Country[k]
cities$city[i] <- world.cities$City[k]
cities$accent_city[i] <- world.cities$AccentCity[k]
cities$dist[i] <- mat[i,k]
}
{% endhighlight %}

Now that this is done, we can print our new data frame to see how it worked.


<!-- html table generated in R 3.2.1 by xtable 1.8-2 package -->
<!-- Sun Jul 24 23:07:51 2016 -->
<table border=1>
<tr> <th>  </th> <th> lat </th> <th> lon </th> <th> country </th> <th> city </th> <th> accent_city </th> <th> dist </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> -33.87 </td> <td align="right"> 151.21 </td> <td> au </td> <td> pyrmont </td> <td> Pyrmont </td> <td align="right"> 0.43 </td> </tr>
  <tr> <td align="right"> 2 </td> <td align="right"> 51.51 </td> <td align="right"> -0.13 </td> <td> gb </td> <td> charing cross </td> <td> Charing Cross </td> <td align="right"> 0.25 </td> </tr>
  <tr> <td align="right"> 3 </td> <td align="right"> 55.75 </td> <td align="right"> 37.50 </td> <td> ru </td> <td> shelepikha </td> <td> Shelepikha </td> <td align="right"> 0.65 </td> </tr>
  <tr> <td align="right"> 4 </td> <td align="right"> 39.91 </td> <td align="right"> 116.40 </td> <td> cn </td> <td> beijing </td> <td> Beijing </td> <td align="right"> 1.55 </td> </tr>
  <tr> <td align="right"> 5 </td> <td align="right"> 35.47 </td> <td align="right"> -97.52 </td> <td> us </td> <td> oklahoma city </td> <td> Oklahoma City </td> <td align="right"> 0.02 </td> </tr>
  <tr> <td align="right"> 6 </td> <td align="right"> -1.28 </td> <td align="right"> 36.82 </td> <td> ke </td> <td> nairoba </td> <td> Nairoba </td> <td align="right"> 0.00 </td> </tr>
  <tr> <td align="right"> 7 </td> <td align="right"> 52.52 </td> <td align="right"> 13.41 </td> <td> de </td> <td> berlin </td> <td> Berlin </td> <td align="right"> 0.74 </td> </tr>
  <tr> <td align="right"> 8 </td> <td align="right"> -22.90 </td> <td align="right"> -43.21 </td> <td> br </td> <td> praia formosa </td> <td> Praia Formosa </td> <td align="right"> 0.16 </td> </tr>
   </table>



## Reverse Geocode US Zipcodes

The reverse Geocode of US Zipcodes is very similar to the world cities algorithm.  Note that this data frame can also be used to geocode data that has a zip code but does not contain lat/lon.  Several open source zip code data files are available online.  The one that I'm going to use here was accessed at [OpenGeoCode](http://www.opengeocode.org/download.php#cities).

First, let's read the data into our environment.  Note that in this case, I explicitly declare the field types so that the zipcode field is read into R as a character field instead of of a numeric field (if read in as a numeric field, it will remove any leading zero's).


{% highlight r %}
zip <- read.csv("cityzip.csv" ,as.is=TRUE,
                colClasses = c("character","character","character","numeric","numeric","numeric"))
{% endhighlight %}






Here's the list of US cities that we will use to test our algorithm.  These lat/lon combinations represent the city center of Portland, Salt Lake City, Dallas, Knowxville, Oklahoma City, Orlando, Raleigh, and Boston.


{% highlight r %}
us.cities <- data.frame(
  lat=c(45.50786 ,40.76078 ,32.78306 ,35.96064 ,35.46756 ,28.53834 ,35.88904 ,42.35843),
  lon=c(-122.69079,-111.89105,-96.80667,-83.92074 , -97.51643 , -81.37924,-82.83104  ,-71.05977)
)
{% endhighlight %}

Next we use an algorithm very similar to the world city algorithm to reverse geocode the lat/lon combinations.  


{% highlight r %}
library(fields)
mat<-rdist.earth(data.matrix(us.cities[,c(2,1)]),data.matrix(zip[,c(5,4)]))

us.cities$city <- NA
us.cities$postal <- NA
us.cities$dist <- NA

for(i in 1:nrow(us.cities)){
k<-which.min(mat[i,])
us.cities$city[i] <- zip$City[k]
us.cities$postal[i] <- zip$Postal[k]
us.cities$dist[i] <- mat[i,k]
}
{% endhighlight %}

Let's check our final product:



   **latitude** |     **longitude** | **city**           | **postal code** | **distance**
:--------------|:---------------:|:--------------:|:-------------:|-------------:
 45.51| -122.69|Portland       |97201  | 0.45
 40.76| -111.89|Salt Lake City |84114  | 0.42
 32.78|  -96.81|Irving         |75202  | 0.04
 35.96|  -83.92|Knoxville      |37902  | 0.21
 35.47|  -97.52|Oklahoma City  |73102  | 0.55
 28.54|  -81.38|Orlando        |32801  | 0.89
 35.89|  -82.83|Hot Springs    |28743  | 0.23
 42.36|  -71.06|Boston         |02109  | 0.51

<table border=1>
<tr> <th>  </th> <th> lat </th> <th> lon </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> -33.87 </td> <td align="right"> 151.21 </td> </tr>
  <tr> <td align="right"> 2 </td> <td align="right"> 51.51 </td> <td align="right"> -0.13 </td> </tr>
  <tr> <td align="right"> 3 </td> <td align="right"> 55.75 </td> <td align="right"> 37.50 </td> </tr>
  <tr> <td align="right"> 4 </td> <td align="right"> 39.91 </td> <td align="right"> 116.40 </td> </tr>
  <tr> <td align="right"> 5 </td> <td align="right"> 35.47 </td> <td align="right"> -97.52 </td> </tr>
  <tr> <td align="right"> 6 </td> <td align="right"> -1.28 </td> <td align="right"> 36.82 </td> </tr>
  <tr> <td align="right"> 7 </td> <td align="right"> 52.52 </td> <td align="right"> 13.41 </td> </tr>
  <tr> <td align="right"> 8 </td> <td align="right"> -22.90 </td> <td align="right"> -43.21 </td> </tr>
   </table>

<table style="width:100%">
  <caption>Monthly savings</caption>
  <tr>
    <th>Month</th>
    <th>Savings</th>
  </tr>
  <tr>
    <td>January</td>
    <td>$100</td>
  </tr>
  <tr>
    <td>February</td>
    <td>$50</td>
  </tr>
</table>

---
layout: post
title:  "Exploring Flickr Data"
categories: [R]
tags: [spatial]
---

In this blog I will explore some geospatial analysis and visualization using the Flickr Data found in the Creative Commons 100M Images/Video.

Introducing Yahoo Flickr Creative Commons 100M Images/Video Data
----------------------------------------------------------------


-   100 Million media objects (picture and Video)
-   Contains objects from 2004 (Flickr inception) through 2014
-   99.2M Photos and 0.8M Video
-   Metadata available
-   69M objects contain tags
-   ~50% of objects are geotagged
-   Available on AWS. See [Yahoo
    Link](https://webscope.sandbox.yahoo.com/catalog.php?datatype=i&did=67)

<img src="https://dmbeskow.github.io/images/2016-08-16-FlickrData/aus_flickr_collage.jpg"  height="300px" />

Plotting point density of Flickr Data in Australia
--------------------------------------------------

Code for creating Point Density Plot of Australia:

    aus.density<-pointdensity(aus.subset,lat_col="lat",lon_col="lon",grid_size=20,radius=50)
    map_base <- qmap('Australia', zoom = 4, darken=0.2) 
    map_base + geom_point(aes(x = lon, y = lat, colour = count,alpha=count), 
        shape = 16, size = 2, data = aus.density) + 
        scale_colour_gradient(low = "yellow", high = "red") 

<table>
<tbody>
<tr class="odd">
<td align="left"><img src="https://dmbeskow.github.io/images/2016-08-16-FlickrData/aus_heat.png" alt="" /></td>
<td align="left"><img src="https://dmbeskow.github.io/images/2016-08-16-FlickrData/sydney_heat.png" alt="" /></td>
</tr>
<tr class="even">
<td align="left">Point Density Plot of Australia</td>
<td align="left">Point Density Plot of Sydney</td>
</tr>
</tbody>
</table>

Introducing Interactive Maps with Leaflet
-----------------------------------------

<center>
<iframe src="https://dmbeskow.github.io/html/leaf_heat.html" width="900" height="600">
</iframe>
</center>
    leaflet(data=sydney) %>% 
      addTiles() %>% addMarkers(~lon, ~lat
    )

Answering questions with data
-----------------------------

-   Can I understand events with this data?
-   Can I understand movement with this data?

Can I understand movement with this data?
-----------------------------------------

    photographers<-unique(sydney$V2)              #ID photographers
    final<-list()                                 #Create OJB to store results
    for(i in 1:length(photographers)){            
    temp<-subset(sydney,V2==photographers[i])     #Subset by photographer
    temp<-temp[order(temp$V4),]                   #Order by time
    breakpoints <- c(FALSE,diff(temp$V4)>8*60)    #Establish break points if diff > 8min
    temp$label <- as.numeric(cut.POSIXt(temp$V4, c(min(temp$V4), 
                  temp[breakpoints, "V4"], max(temp$V4)+1)))    #Convert factor to label
    temp$label<-paste(temp$V2,temp$label,sep="")  #Concatenate label with username
    final[[i]]<-temp                              #Store temp data
    }
    final<-do.call("rbind", final)                #Convert from list to dataframe

<table>
<thead>
<tr class="header">
<th align="left">Flickr Tracks</th>
<th align="left">Understanding the Algorithm</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><img src="https://dmbeskow.github.io/images/2016-08-16-FlickrData/tracks3.png" alt="" /></td>
<td align="left"><img src="https://dmbeskow.github.io/images/2016-08-16-FlickrData/lines.PNG" alt="" /></td>
</tr>
</tbody>
</table>

Crowd sourcing event planning: Best Place to Watch Fireworks
------------------------------------------------------------

<center>
<iframe src="http://data-analytics.net/Apps/fireworks/" width="1020" height="750">
</iframe>
</center>


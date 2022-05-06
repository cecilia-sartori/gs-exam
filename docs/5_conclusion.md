---
layout: page
title: Findings
description: and how I built this
background: '/img/bg-attribution.jpg'
---

From all the previous steps, we  assessed the severity of the problem in Mexico City.
Even though the problem is widespread, violence against women tends to be concentrated in the proximity of the city centre where a high number of people and tourists transit every day. Contextually, the distance to Help Centres in the same area is among the lowest, reveling perhaps some local evidence-based interventions to end violence against women. On the contrarty, reaching a safe place for women living in the suburbs is harder, making those areas more fragile and in need of action. 

# Lessons learned

At first, I tried to use geocoder for plotting data on a map without latitude and longitude (very bad idea).
Geocoding from strings is the best, but when there are a lot of inputs then the job becomes harder. Same goes for boundaries. I ended up having half states with the right geometry and the other with inverted axes. Then I met plotly. 
`OSMnx` is a really great tool for retriving data from OpenStreetMap. However, it works well for small query, such as neighbourhoods analysis. To with entire cities, perhaps it is better to use `pysrom`. 

CRS is not hard to understand, but it took me some time to realize that each type of map wants its own format and that it is not suggested to force another crs (even if allowed). Folium, for instance, allow users to specificy a different CRS, but gave me trouble when trying to plot the  direction route. Finding lat and lon in EPSG:4326 inside the graph node even if reprojected, has been a turning point. 

Despite all the work for preparing data to spatial regression by acquiring socioeconomic variables, I chose not to conduct it for two reason. First, to write the project in a single programming language and second to not overdo.  

Overall, the most time-consuming and demanding part of the project was the *extract, transform, and load* process, starting from researching the data hubs to understanding the structure of the Mexican census.



<span style="white-space: pre"> 

<span style="white-space: pre"> 


# Attribution

For the geospatial analysis, I use the folowing resources:

- [Geospatial Analysis and Representation - UniTN](https://napo.github.io/geospatial_course_unitn/)

- [Geographic Data Science with Python](https://geographicdata.science/book/intro.html) --> this is a very good resource and save my life for Global and Local Spatial Autocorrelation 

- [Spatial data science for sustainable development](https://sustainability-gis.readthedocs.io/en/latest/?badge=latest)

- [Automating GIS-processes 2021](https://autogis-site.readthedocs.io/en/latest/)

- [OSMnx examples - @gboeing](https://github.com/gboeing/osmnx-examples/tree/main/notebooks) --> there is a bunch of good stuff in here. I used it especially for finding the shortest path 


For GitHub page (construction and customization):

- [Anchors to link](https://blog.briandrupieski.com/generate-anchors-in-jekyll-blog-post)

- [Giraffe Academy - Jekyll tutorial](https://youtu.be/T1itpPvFWHI)

- [DataSlice - theme customization](https://youtu.be/wCOInE7-E0I)

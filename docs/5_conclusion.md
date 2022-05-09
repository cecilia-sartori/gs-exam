---
layout: page
title: Findings
description: and attribution
background: '/gs-exam/img/bg-attribution.jpg'
---

From all the previous steps, we  assessed the severity of the problem in Mexico City.
Even though widespread, violence against women tends to be concentrated in the proximity of the city centre where a high number of people and tourists transit every day. Here, the distance to Help Centres is among the shortest, reveling perhaps some government evidence-based interventions. On the contrarty, reaching a safe place for women living in the suburbs is harder, making those areas more fragile and in need of action. 

# Lessons learned

At first, I tried to use geocoder for plotting data on a map without latitude and longitude (very bad idea).
Geocoding from strings is the best, but when there are a lot of inputs then troubles begin. Same goes for boundaries. I ended up having half states with the right geometry and the other with inverted axes and not because of the hemisphere. Then I met `plotly`. 


`OSMnx` is a really great tool for retriving data from OpenStreetMap. However, it works well for small query, such as neighbourhood analysis. With bigger units such as cities, perhaps it is better to use `pysrom`. 

CRS is not hard to understand, but it took me some time to realize that each package for plotting maps wants its own crs format and that even if other formats are supported, better not to use them. `Folium`, for instance, allows users to specificy a different CRS, but gave me trouble when trying to plot the direction route with Mexico City UTM. Finding latitude and longitude in EPSG:4326 inside the reprojected graph node, has been a turning point. 

Despite the effort for preparing data to spatial regression by acquiring socioeconomic variables, I chose not to proceed mainly for keeping the project in a single programming language (I would have use R for this).

Overall, the most time-consuming and demanding part of the project was the *extract, transform, and load* process required for any data set, starting from researching in Mexican data hubs to understanding the structure of the census data.



<span style="white-space: pre"> 

<span style="white-space: pre"> 


# Attribution

All background images are free for commercial use and requires no attribution. See [Pixabay license](https://pixabay.com/service/license/).


**For the geospatial analysis, I used the folowing resources:**

- [Geospatial Analysis and Representation - UniTN](https://napo.github.io/geospatial_course_unitn/)

- [Geographic Data Science with Python](https://geographicdata.science/book/intro.html) --> this is a very good resource and save my life for Global and Local Spatial Autocorrelation

- [GIS workshops - University of California, Los Angeles](https://github.com/yohman)

- [Spatial data science for sustainable development](https://sustainability-gis.readthedocs.io/en/latest/?badge=latest)

- [Automating GIS-processes 2021](https://autogis-site.readthedocs.io/en/latest/)

- [OSMnx examples - @gboeing](https://github.com/gboeing/osmnx-examples/tree/main/notebooks) --> there is a bunch of good stuff in here. I used it especially for finding the shortest path 


**For GitHub page (construction and customization):**

- [GitHub Pages and Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll)

- [Anchors to link](https://blog.briandrupieski.com/generate-anchors-in-jekyll-blog-post)

- [Giraffe Academy - Jekyll tutorial](https://youtu.be/T1itpPvFWHI)

- [DataSlice - theme customization](https://youtu.be/wCOInE7-E0I)

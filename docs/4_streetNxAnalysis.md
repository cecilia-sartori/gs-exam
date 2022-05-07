---
layout: page
title: "Shortest path"
description: "investigating street network"
background: '/gs-exam/img/bg_streetnx.jpg'
---


So far, I have inspected VAW locations to understand their distribution in the urban environment. Imported, cleaned, sorted and plotted with basemaps. Proved the postive spatial correlation and detected which parts of the city have the stronger relationships with the surrounding units.

This project now comes to its last part: investigating the help centre accessibility by answering to the question 'Where is the closest help centre from my place of living?'. 

Detecting the areas with low help centre accessibility can represent an important element in VAW alleviation. 


To meet the goal, I have to retieve the buildings, find the nearest help centre and calculate the shortest path.


Required modules*:

- OSMnx for retrieving buildings and network from Open Street Map and calculating shortest-paths
- Shapely for getting the nearest point


<span style="white-space: pre">

*exclude description of modules imported in the previous steps



```python
# Import modules
import geopandas as gpd
import matplotlib.pyplot as plt
import folium
from folium import plugins

# for network and non analysis
import osmnx as ox
from shapely.ops import nearest_points

# avoid messages
import warnings
warnings.filterwarnings('ignore')
```

## Get buildings footprint from OSM 

`OSMnx` provides the possibility to obtain the building representation from OpenStreetMap by simply entering key-value pairs for tags and name of the place.


```python
tags = {'building': True}
buildings = ox.geometries_from_place("Mexico City, Mexico", tags)
```

Display the outputs:


```python
buildings.plot(markersize=0.2,
               figsize=(12,12))

plt.axis('off') 
plt.show()
```


    
![png](/img/4_streetNxAnalysis/output_6_0.png)
    


It is evident that several buildings are not present on the map. Consequently, the resulting help centre accessibility would be partial and unreliable. In light of this, I prefer to use AGEB geographies. Note that, despite being partial, the number of buildings is high:


```python
len(buildings)
```




    116401



Thus, using these data will be computationally demanding, especially when calculating the nearest center.

## Import AGEB geographies 


```python
# Read the data
gdfa = gpd.read_file('data/VAW_AGEB_clean_lag.gjson')
```

Like buildings, AGEBs are polygons. I have to choose a point of the geometry from which calculate the distance.
In order to get more accurate results, I have to reproject the data to the local UTM.
   
![utm](/img/4_streetNxAnalysis/UTM_CDMX.png)
[source](https://mangomap.com/robertyoung/maps/69585/what-utm-zone-am-i-in-#)

For Mexico City, it is [UTM14N]( https://epsg.io/6369).
```python
gdfa = gdfa.to_crs(epsg=6369) #reproject to CDMX UTM
```

And find the centroid:


```python
gdfa['centroid'] = gdfa.centroid
gdfa.head(2)
```




<div>
<div class="table-wrapper">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CVEGEO</th>
      <th>POBTOT</th>
      <th>PRESOE15</th>
      <th>P5_HLI</th>
      <th>POB_AFRO</th>
      <th>GRAPROES</th>
      <th>PDESOCUP</th>
      <th>PSINDER</th>
      <th>VPH_SINLTC</th>
      <th>VPH_SINCINT</th>
      <th>VAW_COUNT</th>
      <th>VAW_per_1000</th>
      <th>VAW_per_1000_LAG</th>
      <th>geometry</th>
      <th>centroid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0900200010148</td>
      <td>2513.0</td>
      <td>85.0</td>
      <td>16.0</td>
      <td>70.0</td>
      <td>13.26</td>
      <td>39.0</td>
      <td>498.0</td>
      <td>4.0</td>
      <td>90.0</td>
      <td>7</td>
      <td>2.785515</td>
      <td>2.238242</td>
      <td>POLYGON ((478281.827 2156260.877, 478307.065 2...</td>
      <td>POINT (477965.009 2155945.484)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0900200010190</td>
      <td>8920.0</td>
      <td>602.0</td>
      <td>113.0</td>
      <td>218.0</td>
      <td>11.55</td>
      <td>134.0</td>
      <td>2011.0</td>
      <td>76.0</td>
      <td>600.0</td>
      <td>34</td>
      <td>3.811659</td>
      <td>3.507377</td>
      <td>POLYGON ((480069.413 2156118.108, 480081.995 2...</td>
      <td>POINT (479795.113 2155570.427)</td>
    </tr>
  </tbody>
</table>
</div>
</div>



Now import the Help Centre locations.

## Get Help Centre Locations from CDMX Open Data Portal


The data set is licensed under the open data licence (CC BY 4.0).

* **Date source**:

    [Help  Centre for Violence against Women - Mexico City Open Data Portal](https://datos.cdmx.gob.mx/dataset/servicios-de-atencion-a-violencia-contra-las-mujeres/resource/6f553628-ab08-43fe-87a3-810b41464850)



* **Data Structure**:

    Structure of the data is described in a separate excel file ([download here](https://datos.cdmx.gob.mx/dataset/servicios-de-atencion-a-violencia-contra-las-mujeres/resource/d7916650-b9b3-4e4a-9a69-4479a6995724))


```python
# Read the data
gdf = gpd.read_file('data/servicios_de_atencion_a_violencia_contra_las_mujeres/servicios_de_atencion_a_violencia_contra_las_mujeres.shp')
gdf = gdf[gdf['_id'] != 28]
gdf.head(3)
```




<div>
<div class="table-wrapper">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>_id</th>
      <th>id</th>
      <th>id_ente</th>
      <th>id_sede</th>
      <th>nmbr___</th>
      <th>calle</th>
      <th>nmr_xtr</th>
      <th>nmr_ntr</th>
      <th>colonia</th>
      <th>alcaldi</th>
      <th>...</th>
      <th>mdds_d_</th>
      <th>apr____</th>
      <th>infrm__</th>
      <th>prfls_s</th>
      <th>vsts_d_</th>
      <th>a_______</th>
      <th>at______</th>
      <th>i______</th>
      <th>geopont</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>FGJ</td>
      <td>FGJ-005</td>
      <td>Coordinacion Territorial Az-1 y Az-3</td>
      <td>Av de las Culturas y eje 5 Norte</td>
      <td>S/N</td>
      <td>S/N</td>
      <td>San Martin Xochinahuac</td>
      <td>Azcapotzalco</td>
      <td>...</td>
      <td>Sí</td>
      <td>Sí</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>19.5105948,-99.1915796</td>
      <td>POINT (-99.19158 19.51059)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1</td>
      <td>FGJ</td>
      <td>FGJ-007</td>
      <td>Coordinación Territorial BJ-3 y Bj-4</td>
      <td>Avenida Cuauhtémoc esquina Obrero Mundial</td>
      <td>S/N</td>
      <td>S/N</td>
      <td>Navarte</td>
      <td>Benito Juárez</td>
      <td>...</td>
      <td>Sí</td>
      <td>Sí</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>19.4018736,-99.1556199</td>
      <td>POINT (-99.15562 19.40187)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2</td>
      <td>FGJ</td>
      <td>FGJ-010</td>
      <td>Coordinación Territorial Coy-2</td>
      <td>Zompantitla Esq. Tecualiapan</td>
      <td>S/N</td>
      <td>S/N</td>
      <td>Romero de Terreros</td>
      <td>Coyoacán</td>
      <td>...</td>
      <td>Sí</td>
      <td>Sí</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>19.3444204,-99.1666016</td>
      <td>POINT (-99.16660 19.34442)</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 43 columns</p>
</div>
</div>


Reproject to Mexico City UTM:


```python
gdf = gdf.to_crs(epsg=6369)
```

# Find Nearest Point 

My query is one-to-many, namely from the centrid to all possible help centres in the city. So, I need to combine all the help centre locations in a sigle list.


```python
list_points = gdf["geometry"].unary_union
```

And now look for the match. Given the number of points, the following operation will take some time.
Use `shapely - nearest_points` to find the closest geometry. Note that the output will refer to the liner distance.
Retrieve the name of the centre too for future map displaying.


```python
%%time
points = []
link_value = []

for i in range(len(gdfa)):
    nearest_geoms = nearest_points(gdfa.centroid[i], list_points)
    points.append(nearest_geoms[1])
    points_data = gdf.loc[gdf["geometry"] == points[i]]
    link_value.append(points_data['nmbr___'].values[0])
    i += 1
```

    CPU times: user 48.1 s, sys: 0 ns, total: 48.1 s
    Wall time: 48.1 s


Attach outputs to the AGEB geoDataFrame:


```python
gdfa['closest_centre'] =  gpd.GeoSeries(points, crs="EPSG:6369") #ensure geom format
gdfa['Name_centre'] = link_value
```


```python
gdfa.info()
```

    <class 'geopandas.geodataframe.GeoDataFrame'>
    RangeIndex: 2279 entries, 0 to 2278
    Data columns (total 17 columns):
     #   Column            Non-Null Count  Dtype   
    ---  ------            --------------  -----   
     0   CVEGEO            2279 non-null   object  
     1   POBTOT            2279 non-null   float64 
     2   PRESOE15          2279 non-null   float64 
     3   P5_HLI            2279 non-null   float64 
     4   POB_AFRO          2279 non-null   float64 
     5   GRAPROES          2279 non-null   float64 
     6   PDESOCUP          2279 non-null   float64 
     7   PSINDER           2279 non-null   float64 
     8   VPH_SINLTC        2279 non-null   float64 
     9   VPH_SINCINT       2279 non-null   float64 
     10  VAW_COUNT         2279 non-null   int64   
     11  VAW_per_1000      2279 non-null   float64 
     12  VAW_per_1000_LAG  2279 non-null   float64 
     13  geometry          2279 non-null   geometry
     14  centroid          2279 non-null   geometry
     15  closest_centre    2279 non-null   geometry
     16  Name_centre       2279 non-null   object  
    dtypes: float64(11), geometry(3), int64(1), object(2)
    memory usage: 302.8+ KB


At this point, compute the distance in meters using `geopands distance` and convert to km for better understanding.


```python
gdfa['distance'] = gdfa.centroid.distance(gdfa.closest_centre)
gdfa['dist_km'] = gdfa['distance'] /1000
```


```python
gdfa['dist_km'].describe()
```




    count    2279.000000
    mean        1.169192
    std         0.843243
    min         0.007343
    25%         0.601612
    50%         0.985613
    75%         1.500619
    max         6.715206
    Name: dist_km, dtype: float64



On average, the distance from AGEB central points to the help centers is approximately 1.5 km, which is not bad, and the maximum distance from a women in danger to a safe place is less than 7 Km. Display distance on a choroplet map and compare it with the spatial lag map.  



```python
f, axs = plt.subplots(1, 2, figsize=(15, 12))
ax1, ax2 = axs

gdfa.plot(
    column='VAW_per_1000_LAG',
    cmap='RdYlGn_r',
    scheme='quantiles',
    k=5,
    edgecolor='white',
    linewidth=0.,
    alpha=0.75,
    legend=True,
    ax=ax1
)
ax1.set_axis_off()
ax1.set_title("Rates VAW - LAG")


gdfa.plot(
    column='dist_km',
    cmap='RdYlGn_r',
    scheme='quantiles',
    k=5,
    edgecolor='white',
    linewidth=0.,
    alpha=0.75,
    legend=True,
    ax=ax2
)
ax2.set_axis_off()
ax2.set_title("Distance to Help Centres (Km)")


plt.show()
```


    
![png](/img/4_streetNxAnalysis/output_32_0.png)
    


As in many cities around the world, AGEBs on the outskirts of the city are those with less access to services. The distance values we have found are widely acceptable, but do they correspond to reality?

# Get street network with OSMnx


As for neighbourhoods, `OSMnx` allows to download street network data, project and plot the networks. 

It takes the name of the place, and returns the network within its boundary. For future calculation, I prefer to retrieve the driving network type. 


```python
G = ox.graph_from_place('Mexico City, Mexico', network_type='drive')
```

As for the distance between AGEBs, the same story applies for the network. With `ox.project_graph`, the graph will be automatically projected to the local UTM zone.


```python
G_proj = ox.project_graph(G)
```

Since I want to compute distance and travel time from a point A to a point B, I need to impute missing speeds and travel times:


```python
G_proj = ox.speed.add_edge_speeds(G_proj)
G_proj = ox.speed.add_edge_travel_times(G_proj)
```

**Point A, Point B**

I could pick any row, no matter which. I opt for the one with the greatest distance


```python
gdfa.loc[gdfa["dist_km"] == max(gdfa.dist_km)]
```




<div>
<div class="table-wrapper">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CVEGEO</th>
      <th>POBTOT</th>
      <th>PRESOE15</th>
      <th>P5_HLI</th>
      <th>POB_AFRO</th>
      <th>GRAPROES</th>
      <th>PDESOCUP</th>
      <th>PSINDER</th>
      <th>VPH_SINLTC</th>
      <th>VPH_SINCINT</th>
      <th>VAW_COUNT</th>
      <th>VAW_per_1000</th>
      <th>VAW_per_1000_LAG</th>
      <th>geometry</th>
      <th>centroid</th>
      <th>closest_centre</th>
      <th>Name_centre</th>
      <th>distance</th>
      <th>dist_km</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>421</th>
      <td>0901200261710</td>
      <td>785.0</td>
      <td>14.0</td>
      <td>22.0</td>
      <td>38.0</td>
      <td>9.95</td>
      <td>6.0</td>
      <td>328.0</td>
      <td>6.0</td>
      <td>90.0</td>
      <td>6</td>
      <td>7.643312</td>
      <td>2.410339</td>
      <td>POLYGON ((479531.480 2125207.628, 479656.893 2...</td>
      <td>POINT (479491.354 2124303.978)</td>
      <td>POINT (478033.049 2130858.925)</td>
      <td>Hospital General  Ajusco Medio</td>
      <td>6715.206217</td>
      <td>6.715206</td>
    </tr>
  </tbody>
</table>
</div>
</div>


To compute distance in OSM network, I need to find the closest nodes (of the network) to my points, `gdfa.centroid` and `gdfa.closest_centre`.


```python
#specify index of the selected points
i = 421

orig_node = ox.distance.nearest_nodes(G_proj, X=gdfa.centroid[i].x, Y=gdfa.centroid[i].y)
dest_node = ox.distance.nearest_nodes(G_proj, X=gdfa.closest_centre[i].x, Y=gdfa.closest_centre[i].y)
```

Once nodes are found, time to calculate the routes using two different parameters: length and travel time.


```python
route1 = ox.shortest_path(G_proj, orig_node, dest_node, weight="length")
route2 = ox.shortest_path(G_proj, orig_node, dest_node, weight="travel_time")
```

Compare the routes:


```python
# import datetime to convert seconds
import datetime

route1_length = int(sum(ox.utils_graph.get_route_edge_attributes(G_proj, route1, "length")))
route2_length = int(sum(ox.utils_graph.get_route_edge_attributes(G_proj, route2, "length")))

route1_time = int(sum(ox.utils_graph.get_route_edge_attributes(G_proj, route1, "travel_time")))
route2_time = int(sum(ox.utils_graph.get_route_edge_attributes(G_proj, route2, "travel_time")))


print("Route 1 is", route1_length/1000, "km and takes", datetime.timedelta(seconds =route1_time), "to get there by car.")
print("Route 2 is", route2_length/1000, "km and takes", datetime.timedelta(seconds =route2_time), "to get there by car.")
```

    Route 1 is 10.645 km and takes 0:18:43 to get there by car.
    Route 2 is 14.138 km and takes 0:17:46 to get there by car.


Route 2 is longer but smoother, resulting in shorter travel time.

Take a look at the distances. These values are rather out of the linear distance range, for which we had 7 km as maximum value. This tell us that linear distance is good to have a general insight on data, but not suitable for accurate analysis.

## Plotting routes on Networks

Plot the shorter route (for travel time) in green and the longer in red. 


```python
# plot static map
fig, ax = ox.plot_graph_routes(
    G_proj, routes=[route1, route2], route_colors=["r", "g"], route_linewidth=6, node_size=0, 
    bgcolor='w', edge_linewidth=0.2)

```


    
![png](/img/4_streetNxAnalysis/output_50_0.png)
    


Great! It is easy to understand the best route. However, it is impossible to say which direction to take.
We can solve this by plotting the route in `folium`.

First, define a function that returns the shortest route for travel time.


```python
def shortest_time_route(rt1,rt2,r1,r2):
    if rt1 <= rt2:
        return r1
    else:
        return r2
```

Store the shortest route in 'route' variable.


```python
route = shortest_time_route(route1_time,route2_time,route1,route2)
```

The route is nothing but the sequence of OSM nodes. Let's take a look to one node:


```python
G_proj.nodes[orig_node]
```




    {'y': 2124263.8605110194,
     'x': 479429.4467834827,
     'street_count': 3,
     'lon': -99.1956934,
     'lat': 19.211704}



Inside every node, we find both the reprojected coordinates, and in the "original" CRS, namely EPSG:4326.
Use this last to build the folium map. First, create a list of the route nodes.


```python
path = []

for nds in route:
    lon = G_proj.nodes[nds]['lon'] 
    lat = G_proj.nodes[nds]['lat'] 
    path.append([lat, lon])
    nds += 1
```

Note that latitude and longitude are inverted in non-reproject network!


```python
# epsg:3426
G.nodes[orig_node]
```




    {'y': 19.211704, 'x': -99.1956934, 'street_count': 3}



Create the map:


```python

start_location=[G_proj.nodes[orig_node]['lat'], G_proj.nodes[orig_node]['lon']]
direction_m = folium.Map(location = start_location, zoom_start = 12)

#Add orig marker
folium.Marker(
  location=[G_proj.nodes[orig_node]['lat'], G_proj.nodes[orig_node]['lon']],
  tooltip='Origin',
).add_to(direction_m)

#add dest marker
folium.Marker(
  location=[G_proj.nodes[dest_node]['lat'], G_proj.nodes[dest_node]['lon']],
  popup= gdfa['Name_centre'].values[i],
  tooltip='Destination',
    icon=folium.Icon(color='red')
).add_to(direction_m)

folium.plugins.MiniMap().add_to(direction_m)

folium.plugins.AntPath(path).add_to(direction_m)


#direction_m.save("output/shortest_path.html") #save to a file
#direction_m
```


<iframe src="https://cecilia-sartori.github.io/gs-exam/shortest-path/" height="500px" width="100%"></iframe>

[Fullscreen Map](https://cecilia-sartori.github.io/gs-exam/shortest-path/)

Click on destination icon to get the name of the help centre. 

The *snake flow* tells the direction.


<p align="right">
    <a class="btn btn-light" href="{{"/5_conclusion" | relative_url }}" role="button">Next</a>
</p>
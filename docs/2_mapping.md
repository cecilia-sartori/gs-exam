---
layout: page
title: "Display VAW"
description: Get visual intuitions from data
background: '/img/bg_mapping.jpg'
---


First, I need the geography of the city. The smaller the geography, the better the spatial data analysis will be. For this task, I use the Basic Unit of the National Geostatistical Framework (AGEB) that provides a small enough  boundary layer. Additionally, working with AGEB will be useful for future analyses that include census data, since it represents the common framework for any data sets of the Mexican Institute of Statistics (INEGI). 


## Get Block Groups Data from Mexican National Geoinformation Portal

 The portal has several options for formats to export. In this case, I select Shapefile format.

The data set is licensed under the Mexican open data licence (CC BY-NC 2.5 MX).

* **Date source**:

    [Área geoestadística básica urbana, 2019 - Portal de Geoinformación 2022](http://geoportal.conabio.gob.mx/metadatos/doc/html/agbumge19gw.html)

* **Data Structure**:

  Structure of the data is described in a separate pdf file [link here](/pdf/contenido.pdf)


<span style="white-space: pre"> 
For this part, I'll need the following modules:

* Pandas for handling csv files

* Geopandas for handling spatial data

* Matplotlib for plotting figures

* Contextily for adding basemaps
* Plotly for interactive bar chart

* Folium for plotting interactive maps based on leaflet.js


```python
# Import required modules
import pandas as pd
import geopandas as gpd
import matplotlib.pyplot as plt
import contextily as ctx
import plotly.express as px
import folium
from folium import plugins
```


```python
# Read the data
gdf = gpd.read_file('data/ageb/agbumge19gw.shp')
gdf.sample(2)
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
      <th>CVE_ENT</th>
      <th>CVE_MUN</th>
      <th>CVE_LOC</th>
      <th>CVE_AGEB</th>
      <th>NOM_ENT</th>
      <th>NOM_MUN</th>
      <th>AREA</th>
      <th>PERIMETER</th>
      <th>COV_</th>
      <th>COV_ID</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1890</th>
      <td>211140001645A</td>
      <td>21</td>
      <td>114</td>
      <td>0001</td>
      <td>645A</td>
      <td>Puebla</td>
      <td>Puebla</td>
      <td>0.882806</td>
      <td>6.957708</td>
      <td>1890</td>
      <td>1891</td>
      <td>POLYGON ((-98.26667 18.98986, -98.26667 18.989...</td>
    </tr>
    <tr>
      <th>35149</th>
      <td>1510400010088</td>
      <td>15</td>
      <td>104</td>
      <td>0001</td>
      <td>0088</td>
      <td>MÃ©xico</td>
      <td>Tlalnepantla de Baz</td>
      <td>0.469979</td>
      <td>5.239753</td>
      <td>35149</td>
      <td>35150</td>
      <td>POLYGON ((-99.21126 19.56415, -99.21127 19.564...</td>
    </tr>
  </tbody>
</table>
</div>
</div>



CVEGEO is the concatenated geostatistical key that summarises all subsequent elements. This key will be crucial for merging AGEB data with other sources. 

Keep only Mexico City by using the corresponding federal entity key, equal to `09` ([source](/pdf/municipios.pdf)).


```python
gdf = gdf[gdf['CVE_ENT'] == '09']
```

Proceed by filtering the data to only include the relevant columns, namely `CVEGEO` and `geometry`.


```python
gdf = gdf[['CVEGEO', 'geometry']]
```

Inspect the geometries by plotting with matplotlib.  


```python
fig, ax = plt.subplots(figsize=(12,12))

gdf.plot(ax=ax)

plt.axis('off') 
plt.show()
```



![city](/img/2_mapping/output_10_0.png)
    


Is this Mexico City? 
To check it, add a basemap. 
Running the code in the notebook, this can be done easily with the following line of code.


```python
gdf.explore()
```

But nothing is displayed outside the jupyter environment. I use `contextily` to add a basemap to the static map of the geometries.

This package assumes that the Coordinate Reference Systems of the data is in Spherical Mercator (EPSG:3857), unless other crs is specified ([documentation](https://contextily.readthedocs.io/en/latest/reference.html)). 

So, first check CRS format of our data.

**Dealing With Coordinate Reference Systems (CRS)**


```python
# check the CRS
gdf.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



As can be seen above, the CRS is in EPSG:4326. So, to plot the basemap, I can either specify the parameter or reproject the data into EPSG 3857. 

Overall, EPSG 3857 is the preferred CRS for web maps, so I opt for reprojecting.



```python
# reproject to web mercator
gdf = gdf.to_crs(epsg=3857)
```

And plot the city geometries with the basemap.


```python
fig, ax = plt.subplots(figsize=(12,12))

gdf.plot(ax=ax,
         color='black', 
         edgecolor='black',
         lw=0.5,
         alpha=0.2,
         )

ax.axis('off')

# add basemap
ctx.add_basemap(ax,source=ctx.providers.OpenStreetMap.Mapnik)

```



![cdmx_18](/img/2_mapping/output_19_0.png)
    


Now that the spatial information is proven to be correct, let's enrich the geometries with the latest census of population and housing (2020). This makes the data much more informative about the socioeconomic status of the people living in each block and provides the explanatory variables for futere spatial regression analysis.

## Get data of 2020 Census of Population and Housing


The data set is licensed under the Mexican open data licence (CC BY-NC 2.5 MX).

* **Date source**:

    [Censo de Población y Vivienda 2020 - INEGI](https://www.gob.mx/inea/documentos/resultados-del-censo-de-poblacion-y-vivienda-2020-inegi)
    
    

* **Data Structure**:

    Structure of the data is described in a separate pdf file ([link here]())
    
    
    

The data below have been roughly cleaned before importing. See the step [here](/0_datasets/#census-data). 


```python
df = pd.read_csv('data/census_data2020.csv')
df.head()
```




<div>
<div class="divScroll">
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
      <th>ENTIDAD</th>
      <th>NOM_ENT</th>
      <th>MUN</th>
      <th>NOM_MUN</th>
      <th>LOC</th>
      <th>NOM_LOC</th>
      <th>AGEB</th>
      <th>MZA</th>
      <th>POBTOT</th>
      <th>...</th>
      <th>VPH_TELEF</th>
      <th>VPH_CEL</th>
      <th>VPH_INTER</th>
      <th>VPH_STVP</th>
      <th>VPH_SPMVPI</th>
      <th>VPH_CVJ</th>
      <th>VPH_SINRTV</th>
      <th>VPH_SINLTC</th>
      <th>VPH_SINCINT</th>
      <th>VPH_SINTIC</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0900200010010</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>1</td>
      <td>Total AGEB urbana</td>
      <td>0010</td>
      <td>0</td>
      <td>3183.0</td>
      <td>...</td>
      <td>741.0</td>
      <td>772.0</td>
      <td>692.0</td>
      <td>313.0</td>
      <td>221.0</td>
      <td>145.0</td>
      <td>8.0</td>
      <td>14.0</td>
      <td>148.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0900200010025</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>1</td>
      <td>Total AGEB urbana</td>
      <td>0025</td>
      <td>0</td>
      <td>5593.0</td>
      <td>...</td>
      <td>1373.0</td>
      <td>1510.0</td>
      <td>1203.0</td>
      <td>478.0</td>
      <td>349.0</td>
      <td>238.0</td>
      <td>28.0</td>
      <td>68.0</td>
      <td>393.0</td>
      <td>14.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>090020001003A</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>1</td>
      <td>Total AGEB urbana</td>
      <td>003A</td>
      <td>0</td>
      <td>4235.0</td>
      <td>...</td>
      <td>965.0</td>
      <td>1049.0</td>
      <td>878.0</td>
      <td>361.0</td>
      <td>339.0</td>
      <td>247.0</td>
      <td>5.0</td>
      <td>12.0</td>
      <td>250.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0900200010044</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>1</td>
      <td>Total AGEB urbana</td>
      <td>0044</td>
      <td>0</td>
      <td>4768.0</td>
      <td>...</td>
      <td>1124.0</td>
      <td>1237.0</td>
      <td>1076.0</td>
      <td>481.0</td>
      <td>452.0</td>
      <td>294.0</td>
      <td>10.0</td>
      <td>17.0</td>
      <td>254.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0900200010097</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>1</td>
      <td>Total AGEB urbana</td>
      <td>0097</td>
      <td>0</td>
      <td>2176.0</td>
      <td>...</td>
      <td>517.0</td>
      <td>562.0</td>
      <td>507.0</td>
      <td>276.0</td>
      <td>260.0</td>
      <td>153.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>70.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 231 columns</p>
</div>
</div>


As can be seen, census dataset comes with a lot of information. Column headers are defined in the data structure file mentioned above. Here below find the description of what I consider interesting to keep. Drop the rest.


<div class="table-wrapper">
| CODE         | CATEGORY   | DESCRIPTION  |
|--------------|-----------|:-----------|
| **CVEGEO** | GEO | Geostatistical key|
| **POBTOT**| POPULATION | Total number of people habitually living in the federal entity |
| **PRESOE15**  | MIGRATION | Population from 5 to 130 years of age who in 2015 resided in another federal entity |
| **P5_HLI** | ETNICITY | Population from 5 to 130 years of age who speak an indigenous language |
| **POB_AFRO** | ETNICITY | Population that considers itself Afro-Mexican or Afro-descendant |
| **GRAPROES** | EDUCATION | Average degree of education |
| **PDESOCUP** | OCCUPATION | Population of 12 years and more unemployed |
| **PSINDER** | HEALTH | Population with no access to health care |
| **VPH_SINLTC**| DIGITAL ACCESS | Homes with no landline or cell phone  |
| **VPH_SINCINT** | DIGITAL ACCESS | Homes with no computer or Internet |
</div>

Transfer census attributes to AGEB GeoDataFrame by merging on the geostatistical key. Put gdf on the left to keep spatial format in the resulting output.


```python
gdf = pd.merge(gdf, df[['CVEGEO', 'POBTOT', 'PRESOE15',
                        'P5_HLI', 'POB_AFRO', 'GRAPROES',
                        'PDESOCUP', 'PSINDER', 'VPH_SINLTC' ,
                        'VPH_SINCINT']], on="CVEGEO")
```

Some AGEB unit are very small and might refer to few or no people. Get rid of blocks groups with less than 100 total population to further clean the data.


```python
gdf = gdf[gdf['POBTOT']>100]
```

Finally, retrieve the data about violence against women from the Mexico City's Open Data Portal.

### Get VAW Data from CMDX Open Data Portal

VAW data of Mexico City is included in the crime victim dataset. This means that I have to identify which rows are significant to the project based on the description of the crime and the victim's gender.

The data set is licensed under the open data licence (CC BY 4.0).

* **Date source**:

    [Crime victims complaints - Mexico City Open Data Portal](https://datos.cdmx.gob.mx/dataset/victimas-en-carpetas-de-investigacion-fgj)



* **Data Structure**:

    Structure of the data is described in a separate excel file ([download here](https://datos.cdmx.gob.mx/dataset/victimas-en-carpetas-de-investigacion-fgj/resource/10235569-f4a9-4876-9465-9780887df8e2))

**This datasets have been extensively pre-processed [here](/0_datasets/#vaw-data)**
```python
# Read data
vaw = pd.read_csv('data/vaw_filtered.csv')
```

**Exploring Data: Types of violence against women in Mexico City**

Most violence against women is perpetrated by current or former intimate partner and occurs in domestic settings, according to UN ([source](https://www.who.int/publications/i/item/9789240022256a)). Find out what type occurs most in CDMX using an interactive bar plot.


```python
# create a copy to compute required values for the plot
data = vaw.copy() 

# convert time to a datetime64 type 
data['Time'] = pd.to_datetime(data['Time'])
data['Month'] = data['Time'].dt.month # extract the numeric value of months

# group crime by type and count occurrence
occur_type = pd.DataFrame(data.groupby(['Crime', 'Month']).size())
occur_type = occur_type.rename(columns = {0:'Reported'}) # rename the column

# order from Jan to Dec
occur_type.sort_values(["Month"],axis=0, ascending=True, inplace=True)
occur_type.reset_index(inplace = True) # reset index

occur_type.tail(5)
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
      <th>Crime</th>
      <th>Month</th>
      <th>Reported</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>69</th>
      <td>FEMICIDE</td>
      <td>12</td>
      <td>4</td>
    </tr>
    <tr>
      <th>70</th>
      <td>DOMESTIC VIOLENCE</td>
      <td>12</td>
      <td>661</td>
    </tr>
    <tr>
      <th>71</th>
      <td>AGAINST SEXUAL INTIMACY</td>
      <td>12</td>
      <td>26</td>
    </tr>
    <tr>
      <th>72</th>
      <td>SEXUAL ABUSE</td>
      <td>12</td>
      <td>76</td>
    </tr>
    <tr>
      <th>73</th>
      <td>SEXUAL HARASSMENT</td>
      <td>12</td>
      <td>26</td>
    </tr>
  </tbody>
</table>
</div>
</div>



```python
# and plot it
fig = px.bar(occur_type, x="Month", y='Reported',
             color="Crime",
             hover_data={'Crime' : True ,
                         'Month' : False
                               },
             # costumize colours
             color_discrete_map={"DOMESTIC VIOLENCE": "darkblue",
                                 "FEMINICIDE": "royalblue",
                                 "SEXUAL HARASSMENT" : "cornflowerblue",
                                 "SEXUAL ABUSE": "deeppink",
                                },
             labels={"Crime" : "Type of violence",
                     "Reported": "Reported violence"},
             title='Reported Violence Against Women in CDMX by Type, 2021')


fig.update_layout(
        legend=dict(
        orientation="h",
        y = - 0.2
        ),
    legend_title_text=''

)


config = dict({'displayModeBar': False})

#fig.write_html("output/VAW_in_CDMX_byType.html",config=config)
#fig.show(config=config)
```

<iframe src="https://cecilia-sartori.github.io/gs-exam/vaw-type/" height="400px" width="100%"></iframe>

Click on the legend entries to hide and show traces. 

The trend stated by UN have proven to be true for Mexico City. It is worth noting that the number of reported violence is stable throughout the year, but decreases significantly in the months of November and December, revealing that perhaps during the holidays it is more difficult for women to report.

Back to VAW dataframe. Print a summary of the data types:


```python
# get the data types
vaw.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 25419 entries, 480286 to 679497
    Data columns (total 4 columns):
     #   Column     Non-Null Count  Dtype  
    ---  ------     --------------  -----  
     0   Time       25419 non-null  object 
     1   Crime      25419 non-null  object 
     2   latitude   25419 non-null  float64
     3   longitude  25419 non-null  float64
    dtypes: float64(2), object(2)
    memory usage: 992.9+ KB


I can't plot this data on a map because no geometry is listed in the data type column. I need to convert the latitude and longitude into appropriate spatial information. 


```python
# convert df to geodataframe
vaw_gdf = gpd.GeoDataFrame(vaw,
                         crs='EPSG:4326', # lat and long are in degree
                         geometry= gpd.points_from_xy(vaw.longitude, vaw.latitude))
vaw_gdf.head(3)
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
      <th>Time</th>
      <th>Crime</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>480286</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.198153</td>
      <td>-99.026277</td>
      <td>POINT (-99.02628 19.19815)</td>
    </tr>
    <tr>
      <th>480289</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.193027</td>
      <td>-99.095006</td>
      <td>POINT (-99.09501 19.19303)</td>
    </tr>
    <tr>
      <th>480302</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.336963</td>
      <td>-99.152265</td>
      <td>POINT (-99.15227 19.33696)</td>
    </tr>
  </tbody>
</table>
</div>
</div>



Finally, I can plot the VAW locations. However, if many points are concentrated in some areas, then the dots overlap and this makes it hard to understand any pattern.

To represents the locations in a more informative way, use hexagonal cells.



```python
# plot hexbins with Matplotlib
fig, ax = plt.subplots(figsize=(12, 12))

hb= ax.hexbin(data=vaw_gdf,
    x = 'longitude',
    y= 'latitude',
    gridsize=50, # size of hexagonal cells
    alpha=0.5, 
    cmap='viridis_r'
)

# no axis
ax.axis('off')
# add bar side
fig.colorbar(hb, aspect=50, pad=0 );

# hexbin takes lat and lon as arguments --> specify crs
ctx.add_basemap(ax,
                source=ctx.providers.OpenStreetMap.Mapnik,
                crs=4326)
```



![cdmx_54](/img/2_mapping/output_56_0.png)
    


The color of each hexbin denotes the number of points in it. 

With this map, I can instantly draw conclusions. For instance, I see that most cases occurs near the city center.

## Combine Data

At this point, I have two GeoDataFrames: one with the AGEB polygons and the other with the points where the violence took place. I just have to join the two pieces of information with a spatial join.

To do that, I need both data sets with the same map projection.


```python
# reproject vaw_gdf to web mercator
vaw_gdf = vaw_gdf.to_crs(epsg=3857)
```

The following code creates a GeoDataFrame that has every vaw record with the corresponding CVEGEO code.


```python
join = gpd.sjoin(vaw_gdf, gdf, how='left')
join.head(3)
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
      <th>Time</th>
      <th>Crime</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>geometry</th>
      <th>index_right</th>
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>480286</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.198153</td>
      <td>-99.026277</td>
      <td>POINT (-11023554.739 2178279.222)</td>
      <td>1089.0</td>
      <td>0900900010308</td>
      <td>3177.0</td>
      <td>43.0</td>
      <td>101.0</td>
      <td>15.0</td>
      <td>10.38</td>
      <td>22.0</td>
      <td>1568.0</td>
      <td>61.0</td>
      <td>290.0</td>
    </tr>
    <tr>
      <th>480289</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.193027</td>
      <td>-99.095006</td>
      <td>POINT (-11031205.607 2177674.892)</td>
      <td>1080.0</td>
      <td>0900900330350</td>
      <td>9142.0</td>
      <td>149.0</td>
      <td>210.0</td>
      <td>27.0</td>
      <td>10.11</td>
      <td>64.0</td>
      <td>2267.0</td>
      <td>115.0</td>
      <td>806.0</td>
    </tr>
    <tr>
      <th>480302</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.336963</td>
      <td>-99.152265</td>
      <td>POINT (-11037579.654 2194648.346)</td>
      <td>199.0</td>
      <td>0900300010450</td>
      <td>6576.0</td>
      <td>155.0</td>
      <td>58.0</td>
      <td>48.0</td>
      <td>13.21</td>
      <td>101.0</td>
      <td>1764.0</td>
      <td>30.0</td>
      <td>201.0</td>
    </tr>
  </tbody>
</table>
</div>
</div>



**What are the blocks with the highest rate of vaw?**

With the hexagonal cells map, I can understand the data distribution at a glance, but nothing about the block groups.

Therefore, create another dataframe that counts violence by their corresponding AGEB.


```python
reported_by_gdf = join.CVEGEO.value_counts().rename_axis('CVEGEO').reset_index(name='VAW_COUNT')
reported_by_gdf.head()
```




<div>
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
      <th>VAW_COUNT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>090150001113A</td>
      <td>66</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0901500010875</td>
      <td>60</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0901500011036</td>
      <td>58</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0900300010785</td>
      <td>57</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0900300010963</td>
      <td>53</td>
    </tr>
  </tbody>
</table>
</div>



And now join it back to the AGEB gdf


```python
gdf= gdf.merge(reported_by_gdf,on='CVEGEO')
gdf.head()
```




<div>
<div class="divScroll">
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
      <th>geometry</th>
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0900200010148</td>
      <td>POLYGON ((-11043669.988 2213993.494, -11043643...</td>
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
    </tr>
    <tr>
      <th>1</th>
      <td>0900200010190</td>
      <td>POLYGON ((-11041773.412 2213843.334, -11041760...</td>
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
    </tr>
    <tr>
      <th>2</th>
      <td>0900200010932</td>
      <td>POLYGON ((-11041855.381 2215121.578, -11041824...</td>
      <td>408.0</td>
      <td>28.0</td>
      <td>10.0</td>
      <td>0.0</td>
      <td>11.74</td>
      <td>0.0</td>
      <td>58.0</td>
      <td>0.0</td>
      <td>14.0</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0900200010237</td>
      <td>POLYGON ((-11043811.972 2213166.667, -11043827...</td>
      <td>7093.0</td>
      <td>300.0</td>
      <td>61.0</td>
      <td>119.0</td>
      <td>11.13</td>
      <td>115.0</td>
      <td>1881.0</td>
      <td>63.0</td>
      <td>434.0</td>
      <td>24</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0900200010595</td>
      <td>POLYGON ((-11044374.238 2211932.937, -11044339...</td>
      <td>5116.0</td>
      <td>251.0</td>
      <td>65.0</td>
      <td>42.0</td>
      <td>10.59</td>
      <td>70.0</td>
      <td>1324.0</td>
      <td>54.0</td>
      <td>426.0</td>
      <td>13</td>
    </tr>
  </tbody>
</table>
</div>
</div>



The GeoDataFrame now has the vaw count column. So I can easily query the data for finding maximum values. 
However, using the absolute count of violence would be misleading because it doesn't take into account the number of people living in the block. To solve this issue, let's compute the vaw rates per 1000 people by AGEB. 


```python
gdf['VAW_per_1000'] = gdf['VAW_COUNT']/gdf['POBTOT']*1000
```


```python
# Write to GeoJSON to avoid recalculation 

gdf.to_file("data/VAW_AGEB_clean.gjson", driver="GeoJSON")
```

And plot the Top 10 Most Dangerous Blocks:


```python
# plot the top 10 geographies
fig,ax = plt.subplots(figsize=(15,15))

gdf.sort_values(by='VAW_per_1000',ascending=False)[:10].plot(ax=ax,
                                                                 color='red',
                                                                 edgecolor='white',
                                                                 alpha=0.5,legend=True)


# title
ax.set_title('Top 10 Most Dangerous Blocks for Women in CDMX, 2021', fontsize=22)

# no axis
ax.axis('off')

# add a basemap
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
```


    
![top10](/img/2_mapping/output_69_0.png)
    


Now visualise the total distribution with a choropleth map:


```python
fig,ax = plt.subplots(figsize=(15,15))

gdf.plot(ax=ax,
        column='VAW_per_1000',
        legend=True,
        alpha=0.6,
        cmap='RdYlGn_r',
        scheme='quantiles')

ax.axis('off')
ax.set_title('Rates of violence against women per 1000 residents in CDMX, 2021',fontsize=22)

ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
```


    
![chpl](/img/2_mapping/output_71_0.png)
    


The fugure above presents information in a simple, visual way. However, it would be much better for users to be free to zoom in some areas and play around, namely if the figure were interactive! I can use `folium` to accomplish the task.

Moreover, AGEB are really useful to understand whether similar geographies tend to cluster (and apparently they do in this case). But what about the district they belong to?
Let's find out the worst neighborhood to be a woman in Mexico City - as claimed by the title of the project - by fisrt importing the Neighborhood Data.


## Get Neighborhood Data

Data comes from Mexico City Open Data Portal and have been cleaned before importig. Find the pre-processign phase [here](/0_datasets/#neighbourhood-data).


```python
# Read the data
gdf_mun = gpd.read_file('data/CDMX_municipalities.gjson')
# reproject to web mercator
gdf_mun = gdf_mun.to_crs(epsg=3857)

gdf_mun.head(3)
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
      <th>NOMGEO</th>
      <th>POBTOT</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9009</td>
      <td>MILPA ALTA</td>
      <td>152685</td>
      <td>POLYGON ((-11020321.650 2181716.432, -11020345...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9014</td>
      <td>BENITO JUÁREZ</td>
      <td>434153</td>
      <td>POLYGON ((-11035857.497 2202270.237, -11035860...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9005</td>
      <td>GUSTAVO A. MADERO</td>
      <td>1173351</td>
      <td>POLYGON ((-11033831.810 2223869.140, -11033643...</td>
    </tr>
  </tbody>
</table>
</div>
</div>



Run the same operations performed before for AGEB.



```python
# Make a spatial join
join = gpd.sjoin(vaw_gdf, gdf_mun, how="left")
# count vaw per neighbourhood
reported_by_gdf = join.CVEGEO.value_counts().rename_axis('CVEGEO').reset_index(name='VAW_COUNT')
```


```python
gdf_mun= gdf_mun.merge(reported_by_gdf,on='CVEGEO')
```


```python
#Compute rate per 1000
gdf_mun['VAW_per_1000'] = gdf_mun['VAW_COUNT']/gdf['POBTOT']*1000
gdf_mun.head()
```




<div>
<div class="divScroll">
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
      <th>NOMGEO</th>
      <th>POBTOT</th>
      <th>geometry</th>
      <th>VAW_COUNT</th>
      <th>VAW_per_1000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9009</td>
      <td>MILPA ALTA</td>
      <td>152685</td>
      <td>POLYGON ((-11020321.650 2181716.432, -11020345...</td>
      <td>508</td>
      <td>202.148826</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9014</td>
      <td>BENITO JUÁREZ</td>
      <td>434153</td>
      <td>POLYGON ((-11035857.497 2202270.237, -11035860...</td>
      <td>904</td>
      <td>101.345291</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9005</td>
      <td>GUSTAVO A. MADERO</td>
      <td>1173351</td>
      <td>POLYGON ((-11033831.810 2223869.140, -11033643...</td>
      <td>3284</td>
      <td>8049.019608</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9003</td>
      <td>COYOACÁN</td>
      <td>614447</td>
      <td>POLYGON ((-11036128.811 2196996.411, -11035960...</td>
      <td>1625</td>
      <td>229.099112</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9016</td>
      <td>MIGUEL HIDALGO</td>
      <td>414470</td>
      <td>POLYGON ((-11041844.793 2210106.057, -11041853...</td>
      <td>1027</td>
      <td>200.742768</td>
    </tr>
  </tbody>
</table>
</div>
</div>


We are now ready to combine the data into a single interactive map.

## Build an Interactive Choropleth Map

Display AGEB and neighbourhood in two FeatureGroup layers map:


```python
#Create map instance, set location
vaw_map =folium.Map(location=[19.32096,-99.15261], 
                   tiles = 'cartodbpositron', 
                   zoom_start=10, control_scale=True, overlay=False)
#set two layer
fg1 = folium.FeatureGroup(name='VAW rate per AGEB',overlay=False).add_to(vaw_map)
fg2 = folium.FeatureGroup(name='VAW rate per Neighbourhood',overlay=False).add_to(vaw_map)

```


```python
#Add the first choropleth map layer to fg1
custom_scale1 = (gdf['VAW_per_1000'].quantile((0,0.2,0.4,0.6,0.8,1))).tolist()

ageb = folium.Choropleth(
        geo_data=gdf,
        name='vaw in 2021',
        data=gdf,
        columns=['CVEGEO', 'VAW_per_1000'],
        key_on='feature.properties.CVEGEO',
        fill_color='YlOrRd',
        fill_opacity=0.7,
        line_opacity=0.2,
        line_color='white', 
        line_weight=0, 
        smooth_factor=1,
        threshold_scale=custom_scale1,
        legend_name= 'Rates of violence against women per 1000 in CDMX, 2021',
    overlay=False).geojson.add_to(fg1)


folium.features.GeoJson(
                    data=gdf,
                    smooth_factor=2,
                    style_function=lambda x: {'color':'white','fillColor':'transparent','weight':0.5},
                    tooltip=folium.features.GeoJsonTooltip(
                        fields=['POBTOT',
                                'VAW_COUNT',
                                'VAW_per_1000',
                                'PDESOCUP',
                                'GRAPROES'
                               ],
                        aliases=["Total Population, <br>(Per AGEB):",
                                 "Reported cases:",
                                 "VAW rate per 1000:",
                                 "Pop Unemployed:",
                                 "Degree of Education:"
                                ], 
                        localize=True,
                        sticky=False,
                        labels=True,
                        style="""
                            background-color: #F0EFEF;
                            border: 2px solid black;
                            border-radius: 3px;
                            box-shadow: 3px;
                        """,
                        max_width=800,),
                            highlight_function=lambda x: {'weight':3,'fillColor':'grey'},
                        ).add_to(ageb)

#Add the 2nd choropleth map layer to fg2
custom_scale2 = (gdf_mun['VAW_per_1000'].quantile((0,0.2,0.4,0.6,0.8,1))).tolist()

neighb = folium.Choropleth(
    geo_data=gdf_mun,
    name='vaw per Neighbourhood',
    data=gdf_mun,
    columns=['CVEGEO', 'VAW_per_1000'],
    key_on='feature.properties.CVEGEO',
    fill_color='YlOrRd',
    fill_opacity=0.7,
    line_opacity=0.2,
    line_color='white', 
    line_weight=0, 
    smooth_factor=1,
    threshold_scale=custom_scale2,
    legend_name= 'Incidents of violence against women in CDMX, 2021',
    overlay=False).geojson.add_to(fg2)

folium.features.GeoJson(
                    data=gdf_mun,
                    smooth_factor=2,
                    style_function=lambda x: {'color':'white','fillColor':'transparent','weight':0.5},
                    tooltip=folium.features.GeoJsonTooltip(
                        fields=['NOMGEO',
                                'POBTOT',
                                'VAW_COUNT',
                                'VAW_per_1000',
                               ],
                        aliases=["Neighbourhood:",
                                 "Population:",
                                 "Reported Cases:",
                                 "VAW rate per 1000:"
                                 
                                ], 
                        localize=True,
                        sticky=False,
                        labels=True,
                        style="""
                            background-color: #F0EFEF;
                            border: 2px solid black;
                            border-radius: 3px;
                            box-shadow: 3px;
                        """,
                        max_width=800,),
                            highlight_function=lambda x: {'weight':3,'fillColor':'grey'},
                        ).add_to(neighb)   


#Add layer control to the map
folium.TileLayer('cartodbdark_matter',overlay=True,name="View in Dark Mode").add_to(vaw_map)
folium.TileLayer('cartodbpositron',overlay=True,name="Viw in Light Mode").add_to(vaw_map)
folium.LayerControl(collapsed=False).add_to(vaw_map)
#Add mini map
folium.plugins.MiniMap().add_to(vaw_map)

#vaw_map.save("output/CDMX_VAW_Folium.html") #save to a file
#vaw_map
```


<iframe src="https://cecilia-sartori.github.io/gs-exam/cdmx-folium/" height="500px" width="100%"></iframe>


Et Voilà! Click on the legend entries to hide and show the layer. 

The AGEB layer visually suggests that there are spatial clusters of where VAW is more prevalent, but can this pattern be a matter of chance?

To answer this question, I need to conduct the spatial correlation analysis.
Go to the next page.

<span style="white-space: pre"> 

<p align="right">
    <a class="btn btn-light" href="{{"/3_spatialCorr" | relative_url }}" role="button">Next</a>
</p>
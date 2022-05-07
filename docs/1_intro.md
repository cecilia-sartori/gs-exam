---
layout: page
title: "VAW in the World"
description: Providing Context
background: '/img/bg_intro.jpg'
---


## Introduction

A [study](https://www.who.int/news/item/09-03-2021-devastatingly-pervasive-1-in-3-women-globally-experience-violence) conducted by the World Health Organization (WHO) in 2018 reveals that inequality is the primary driver of violence. In low-an-middle income countries, women annually report rates of violence that are up to three times greater than those living in the most developed countries.

Let's use the data from the [WHO Global Health Observatory](https://www.who.int/data/gho/indicator-metadata-registry/imr-details/3685) to figure out what countries have the highest prevalence rates of intimate partner violence among women aged 15-49. Consider two indicators: 

- Intimate partner violence prevalence among ever partnered women in their lifetime (%)
- Intimate partner violence prevalence among ever partnered women in the previous 12 months (%) (latest available year: 2018)

### How to plot data on a map without latitude and longitude?

In most cases, the only way is to use a geocoder. It takes strings as input, such as addresses, and returns georeferenced locations. However,there is no free lunch. The names of the places can correspond to more than one location, such as Georgia (USA Federal State and country located in the Caucasus), and territories with controversial international recognition (Kosovo under UNSCR 1244, Palestinian territories, Hong Kong...) are hardly ever decoded correctly. What to do then? In general, providing more information to the geocoder ensures a better result. In the Georgia case, adding subregions to the query works.

In some lucky cases (as this very one), when data have no geometry but the country codes ([ISO](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes)), we can use `plotly`, a fancy library that creates great interactive maps.


Let's begin by importing the required modules:

* Pandas for handling tabular data
* Plotly for plotting interactive maps based on ISO codes


```python
# Import required modules
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
```

Read data:


```python
df12M = pd.read_csv("data/WHO_vaw_previous12months.csv")
dfLFT = pd.read_csv('data/WHO_vaw_lifetime.csv')
```

Merge the two indicators:


```python
df = pd.merge(df12M, dfLFT, on= 'SpatialDimValueCode', how='outer',suffixes=('_last12M','_lifetime'))
df.head(3)
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
      <th>IndicatorCode_last12M</th>
      <th>Indicator_last12M</th>
      <th>ValueType_last12M</th>
      <th>ParentLocationCode_last12M</th>
      <th>ParentLocation_last12M</th>
      <th>Location type_last12M</th>
      <th>SpatialDimValueCode</th>
      <th>Location_last12M</th>
      <th>Period type_last12M</th>
      <th>Period_last12M</th>
      <th>...</th>
      <th>FactValueUoM_lifetime</th>
      <th>FactValueNumericLowPrefix_lifetime</th>
      <th>FactValueNumericLow_lifetime</th>
      <th>FactValueNumericHighPrefix_lifetime</th>
      <th>FactValueNumericHigh_lifetime</th>
      <th>Value_lifetime</th>
      <th>FactValueTranslationID_lifetime</th>
      <th>FactComments_lifetime</th>
      <th>Language_lifetime</th>
      <th>DateModified_lifetime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>SDGIPV12M</td>
      <td>Proportion of ever-partnered women and girls a...</td>
      <td>numeric</td>
      <td>AMR</td>
      <td>Americas</td>
      <td>Country</td>
      <td>DOM</td>
      <td>Dominican Republic</td>
      <td>Year</td>
      <td>2018.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>29.0</td>
      <td>NaN</td>
      <td>13.0</td>
      <td>19 [29 – 13]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>EN</td>
      <td>2021-05-05T21:00:00.000Z</td>
    </tr>
    <tr>
      <th>1</th>
      <td>SDGIPV12M</td>
      <td>Proportion of ever-partnered women and girls a...</td>
      <td>numeric</td>
      <td>AMR</td>
      <td>Americas</td>
      <td>Country</td>
      <td>MEX</td>
      <td>Mexico</td>
      <td>Year</td>
      <td>2018.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>35.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>24 [35 – 16]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>EN</td>
      <td>2021-05-05T21:00:00.000Z</td>
    </tr>
    <tr>
      <th>2</th>
      <td>SDGIPV12M</td>
      <td>Proportion of ever-partnered women and girls a...</td>
      <td>numeric</td>
      <td>WPR</td>
      <td>Western Pacific</td>
      <td>Country</td>
      <td>VNM</td>
      <td>Viet Nam</td>
      <td>Year</td>
      <td>2018.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>38.0</td>
      <td>NaN</td>
      <td>15.0</td>
      <td>25 [38 – 15]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>EN</td>
      <td>2021-05-05T21:00:00.000Z</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 67 columns</p>
</div>
</div>



Create the Choropleth Map: 


```python
fig = px.choropleth(df, locations= 'SpatialDimValueCode', color = 'FactValueNumeric_lifetime',
                    hover_name='Location_lifetime',
                    hover_data={'FactValueNumeric_lifetime' : True ,
                                'FactValueNumeric_last12M' : True,
                                'SpatialDimValueCode' : False
                               },
                    labels={'FactValueNumeric_lifetime':'Lifetime (%)',
                            'FactValueNumeric_last12M' : 'Past Year (%)'
                           }
                    )

fig.update_layout(
    title = go.layout.Title(
        text = 'Women subjected to violence by an intimate partner <br> at least once in their lifetime (%)'),
        title_x=0.5,
        
    hoverlabel=dict(
        bgcolor="white",
        font_size=14,
        font_family="Rockwell"
    ),
    
    geo = go.layout.Geo(
        showframe = True,
        showcountries=True, countrycolor = "white",
        showocean = True, oceancolor = '#c9d2e0',
        coastlinecolor = "white",
        projection_type = 'natural earth'),

)

config = dict({'displayModeBar': False})

#fig.show(config=config)
#fig.write_html("output/VAW_in_World.html",config=config)
```
<iframe src="https://cecilia-sartori.github.io/gs-exam/wav-in-world/" height="500px" width="100%"></iframe>

[Fullscreen Map](https://cecilia-sartori.github.io/gs-exam/wav-in-world/)

States are correctly displayed and data easily turned into informative interactive content.

 As the figure shows, VAWG is widespread with a higher prevalence in least developed countries. However, a recent [UN WOMAN report](https://worlds-women-2020-data-undesa.hub.arcgis.com/) states that the phenomenon is largely underreported, both in stable and emergency contexts, revealing that data collection on the issue is difficult and data themselves often miss the whole picture. 

## A Quick Focus on Mexico

In Mexico, twenty-four per cent of women aged 15 to 49 have been subject to physical and/or sexual intimate partner violence in their life, ten per cent in the past 12 months. Overall, it scores a high rate of violence against women, but similar to countries in the region, confirming that enormous efforts must be made across Latin America and the Caribbean to eradicate violence against women. 

In the next step, I'll focus on how to map the prevalence of violence against women in Mexico City. 


<span style="white-space: pre"> 

<p align="right">
    <a class="btn btn-light" href="{{"/2_mapping" | relative_url }}" role="button">Next</a>
</p>

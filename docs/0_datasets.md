---
layout: page
title: "Pre-Processing"
description: Step-by-step
background: '/gs-exam/img/bg_mapping1.jpg'
---



# Data pre-processing


# Census Data

Data cover the entire Mexico City, but have a sub-unit (Manzana) that we do not consider. After importing, select AGEB, replace special character `*` with Nan values, select non-string objects and convert them into float data type. Save file.


```python
import pandas as pd
import numpy as np
```

INEGI portal, inegi.org.mx, has been down for a while. Use the local version.


```python
df = pd.read_csv('data/RESAGEBURB_09XLSX20.csv')
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
      <td>0900000000000</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>0</td>
      <td>Total de la entidad Ciudad de México</td>
      <td>0</td>
      <td>Total de la entidad</td>
      <td>0000</td>
      <td>0</td>
      <td>9209944</td>
      <td>...</td>
      <td>1898265</td>
      <td>2536523</td>
      <td>2084156</td>
      <td>1290811</td>
      <td>957162</td>
      <td>568827</td>
      <td>46172</td>
      <td>77272</td>
      <td>561128</td>
      <td>10528</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0900200000000</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>0</td>
      <td>Total del municipio</td>
      <td>0000</td>
      <td>0</td>
      <td>432205</td>
      <td>...</td>
      <td>96128</td>
      <td>123961</td>
      <td>105899</td>
      <td>66399</td>
      <td>50965</td>
      <td>31801</td>
      <td>1661</td>
      <td>2869</td>
      <td>22687</td>
      <td>322</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0900200010000</td>
      <td>9</td>
      <td>Ciudad de México</td>
      <td>2</td>
      <td>Azcapotzalco</td>
      <td>1</td>
      <td>Total de la localidad urbana</td>
      <td>0000</td>
      <td>0</td>
      <td>432205</td>
      <td>...</td>
      <td>96128</td>
      <td>123961</td>
      <td>105899</td>
      <td>66399</td>
      <td>50965</td>
      <td>31801</td>
      <td>1661</td>
      <td>2869</td>
      <td>22687</td>
      <td>322</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 231 columns</p>
</div>
</div>



```python
df = df[df['NOM_LOC']=='Total AGEB urbana']
df = df.replace('*', np.nan)
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2433 entries, 3 to 68915
    Columns: 231 entries, CVEGEO to VPH_SINTIC
    dtypes: int64(6), object(225)
    memory usage: 4.3+ MB


Most of the entries are objects. We need to convert them in numbers.


```python
df1 = df.iloc[:,:9]
df2 = df.iloc[:,9:]
df2 = df2.fillna(0).astype(float)
```


```python
df = pd.concat([df1,df2], axis=1)
```


```python
#save file
df.to_csv('data/census_data2020.csv', index = False)
```

# VAW Data

VAW data of Mexico City is included in the crime victim dataset. This means that I have to identify which rows are significant to the project based on the description of the crime and the victim's gender.


```python
# Read data
vaw = pd.read_csv('data/victimas_completa_marzo_2022.csv')
vaw.head()
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
      <th>idCarpeta</th>
      <th>Año_inicio</th>
      <th>Mes_inicio</th>
      <th>FechaInicio</th>
      <th>Delito</th>
      <th>Categoria</th>
      <th>Sexo</th>
      <th>Edad</th>
      <th>TipoPersona</th>
      <th>CalidadJuridica</th>
      <th>...</th>
      <th>Mes_hecho</th>
      <th>FechaHecho</th>
      <th>HoraHecho</th>
      <th>HoraInicio</th>
      <th>AlcaldiaHechos</th>
      <th>ColoniaHechos</th>
      <th>Calle_hechos</th>
      <th>Calle_hechos2</th>
      <th>latitud</th>
      <th>longitud</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8324429.0</td>
      <td>2019.0</td>
      <td>Enero</td>
      <td>04/01/2019</td>
      <td>FRAUDE</td>
      <td>DELITO DE BAJO IMPACTO</td>
      <td>Masculino</td>
      <td>62.0</td>
      <td>FISICA</td>
      <td>OFENDIDO</td>
      <td>...</td>
      <td>Agosto</td>
      <td>29/08/2018</td>
      <td>12:00:00</td>
      <td>12:19:00</td>
      <td>ALVARO OBREGON</td>
      <td>GUADALUPE INN</td>
      <td>INSUGENTES SUR</td>
      <td>NaN</td>
      <td>19.36125</td>
      <td>-99.18314</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8324430.0</td>
      <td>2019.0</td>
      <td>Enero</td>
      <td>04/01/2019</td>
      <td>PRODUCCIÓN, IMPRESIÓN, ENAJENACIÓN, DISTRIBUCI...</td>
      <td>DELITO DE BAJO IMPACTO</td>
      <td>Femenino</td>
      <td>38.0</td>
      <td>FISICA</td>
      <td>VICTIMA Y DENUNCIANTE</td>
      <td>...</td>
      <td>Diciembre</td>
      <td>15/12/2018</td>
      <td>15:00:00</td>
      <td>12:20:00</td>
      <td>AZCAPOTZALCO</td>
      <td>VICTORIA DE LAS DEMOCRACIAS</td>
      <td>AV.  CUATLAHUAC</td>
      <td>NaN</td>
      <td>19.47181</td>
      <td>-99.16458</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8324431.0</td>
      <td>2019.0</td>
      <td>Enero</td>
      <td>04/01/2019</td>
      <td>ROBO A TRANSEUNTE SALIENDO DEL BANCO CON VIOLE...</td>
      <td>ROBO A CUENTAHABIENTE SALIENDO DEL CAJERO CON ...</td>
      <td>Masculino</td>
      <td>42.0</td>
      <td>FISICA</td>
      <td>VICTIMA Y DENUNCIANTE</td>
      <td>...</td>
      <td>Diciembre</td>
      <td>22/12/2018</td>
      <td>15:30:00</td>
      <td>12:23:00</td>
      <td>COYOACAN</td>
      <td>COPILCO UNIVERSIDAD ISSSTE</td>
      <td>COPILCO</td>
      <td>NaN</td>
      <td>19.33797</td>
      <td>-99.18611</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8324435.0</td>
      <td>2019.0</td>
      <td>Enero</td>
      <td>04/01/2019</td>
      <td>ROBO DE VEHICULO DE SERVICIO PARTICULAR SIN VI...</td>
      <td>ROBO DE VEHÍCULO CON Y SIN VIOLENCIA</td>
      <td>Masculino</td>
      <td>35.0</td>
      <td>FISICA</td>
      <td>VICTIMA Y DENUNCIANTE</td>
      <td>...</td>
      <td>Enero</td>
      <td>04/01/2019</td>
      <td>06:00:00</td>
      <td>12:27:00</td>
      <td>IZTACALCO</td>
      <td>AGRÍCOLA PANTITLAN</td>
      <td>CALLE 6</td>
      <td>ENTRE PRIVADA DEL VALLE Y PRIVADA GONZALEZ</td>
      <td>19.40327</td>
      <td>-99.05983</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8324438.0</td>
      <td>2019.0</td>
      <td>Enero</td>
      <td>04/01/2019</td>
      <td>ROBO DE MOTOCICLETA SIN VIOLENCIA</td>
      <td>ROBO DE VEHÍCULO CON Y SIN VIOLENCIA</td>
      <td>Masculino</td>
      <td>NaN</td>
      <td>FISICA</td>
      <td>VICTIMA</td>
      <td>...</td>
      <td>Enero</td>
      <td>03/01/2019</td>
      <td>20:00:00</td>
      <td>12:35:00</td>
      <td>IZTAPALAPA</td>
      <td>PROGRESISTA</td>
      <td>UNIVERSIDAD</td>
      <td>NaN</td>
      <td>19.35480</td>
      <td>-99.06324</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>
</div>



Inspect with info() method to get better data insight.



```python
vaw.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 780439 entries, 0 to 780438
    Data columns (total 22 columns):
     #   Column           Non-Null Count   Dtype  
    ---  ------           --------------   -----  
     0   idCarpeta        780439 non-null  float64
     1   Año_inicio       780438 non-null  float64
     2   Mes_inicio       780438 non-null  object 
     3   FechaInicio      780438 non-null  object 
     4   Delito           780439 non-null  object 
     5   Categoria        780439 non-null  object 
     6   Sexo             628287 non-null  object 
     7   Edad             495406 non-null  float64
     8   TipoPersona      774405 non-null  object 
     9   CalidadJuridica  780438 non-null  object 
     10  competencia      780439 non-null  object 
     11  Año_hecho        780120 non-null  float64
     12  Mes_hecho        780120 non-null  object 
     13  FechaHecho       780120 non-null  object 
     14  HoraHecho        780130 non-null  object 
     15  HoraInicio       780438 non-null  object 
     16  AlcaldiaHechos   779234 non-null  object 
     17  ColoniaHechos    745330 non-null  object 
     18  Calle_hechos     778237 non-null  object 
     19  Calle_hechos2    280532 non-null  object 
     20  latitud          745544 non-null  float64
     21  longitud         745542 non-null  float64
    dtypes: float64(6), object(16)
    memory usage: 131.0+ MB


There are 679 500 observations, which are quite a lot! Notice that the non-null values of latitude and longitude are less than the total entries. Therefore, drop all rows that have a NaN value in these columns and run a preliminary sub-selection by keeping female victims and latest year available.

From the data structure, I know that "Ano_inicio" field corresponds to the opening date of the investigation, while "Ano_hecho" represents the date on which the crime was committed. Then, go for the second.


```python
# drop Nan values
vaw = vaw.dropna(subset=['latitud', 'longitud'])

# sort unique years in the data
year = sorted(vaw['Año_hecho'].unique())
print("Last year available: ", year[-1])
```

    Last year available:  2022.0


Filter data accordingly


```python
vaw = vaw[vaw['Año_hecho']== 2021]
vaw = vaw[vaw['Sexo']=='Femenino']
```

**Abortion in Mexico: a Crime?**

Since 2021, abortion in Mexico is no longer a crime, although its legalisation still varies by state ([source](https://edition.cnn.com/2021/09/07/americas/mexico-criminalizing-abortion-unconstitutional-intl-latam/index.html)). 

Yet, in our data it still labeled as a crime:


```python
vaw[vaw['Delito']== 'ABORTO'].sample(1)
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
      <th>idCarpeta</th>
      <th>Año_inicio</th>
      <th>Mes_inicio</th>
      <th>FechaInicio</th>
      <th>Delito</th>
      <th>Categoria</th>
      <th>Sexo</th>
      <th>Edad</th>
      <th>TipoPersona</th>
      <th>CalidadJuridica</th>
      <th>...</th>
      <th>Mes_hecho</th>
      <th>FechaHecho</th>
      <th>HoraHecho</th>
      <th>HoraInicio</th>
      <th>AlcaldiaHechos</th>
      <th>ColoniaHechos</th>
      <th>Calle_hechos</th>
      <th>Calle_hechos2</th>
      <th>latitud</th>
      <th>longitud</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>566672</th>
      <td>8929948.0</td>
      <td>2021.0</td>
      <td>Mayo</td>
      <td>17/05/2021</td>
      <td>ABORTO</td>
      <td>DELITO DE BAJO IMPACTO</td>
      <td>Femenino</td>
      <td>NaN</td>
      <td>FISICA</td>
      <td>VICTIMA</td>
      <td>...</td>
      <td>Mayo</td>
      <td>17/05/2021</td>
      <td>12:00:00</td>
      <td>17:49:00</td>
      <td>VENUSTIANO CARRANZA</td>
      <td>AMPLIACION ADOLFO LOPEZ MATEOS</td>
      <td>EJE 4 OTE. RIO CHURUBUSCO ESQUINA CON LA CALLE...</td>
      <td>NaN</td>
      <td>19.416515</td>
      <td>-99.073591</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 22 columns</p>
</div>
</div>


Considering abortion as a crime is beyond the scope of this project, but you too may find this interesting. 

**Back to Data Preparation**

Select vaw crimes that match with the strings "sexual, domestic, femicide" and then print the results.


```python
vaw = vaw[vaw['Delito'].str.contains('sexual|familiar|feminicidio', case=False)]

print('List of crimes after text query:\n', vaw['Delito'].unique())
```

    List of crimes after text query:
     ['VIOLENCIA FAMILIAR' 'ABUSO SEXUAL' 'ACOSO SEXUAL'
     'CONTRA LA INTIMIDAD SEXUAL' 'ACOSO SEXUAL AGRAVADO EN CONTRA DE MENORES'
     'FEMINICIDIO' 'FEMINICIDIO POR ARMA BLANCA' 'TENTATIVA DE FEMINICIDIO'
     'FEMINICIDIO POR DISPARO DE ARMA DE FUEGO'
     'PRIVACION DE LA LIBERTAD PERSONAL (REALIZAR ACTO SEXUAL)'
     'FEMINICIDIO POR GOLPES']


Using str.contains() as filter method in data preparation is highly not recommended because the chance of missing important information is real. For example, strings can have special characters. In this case, I didn't find any viable alternative.

Let's now begin to translate the DataFrame from Spanish to English starting from the columns names:


```python
# select columns and rename
vaw = vaw[['FechaHecho', 'Delito', 'latitud', 'longitud']]
vaw.columns = ['Time', 'Crime', 'latitude', 'longitude']
```


```python
vaw
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
      <th>Time</th>
      <th>Crime</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>480286</th>
      <td>01/01/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.198153</td>
      <td>-99.026277</td>
    </tr>
    <tr>
      <th>480289</th>
      <td>01/01/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.193027</td>
      <td>-99.095006</td>
    </tr>
    <tr>
      <th>480302</th>
      <td>01/01/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.336963</td>
      <td>-99.152265</td>
    </tr>
    <tr>
      <th>480306</th>
      <td>01/01/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.561525</td>
      <td>-99.146590</td>
    </tr>
    <tr>
      <th>480320</th>
      <td>01/01/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.395045</td>
      <td>-99.061060</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>779813</th>
      <td>08/12/2021</td>
      <td>ABUSO SEXUAL</td>
      <td>19.282468</td>
      <td>-99.010266</td>
    </tr>
    <tr>
      <th>780165</th>
      <td>31/03/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.371821</td>
      <td>-99.071966</td>
    </tr>
    <tr>
      <th>780166</th>
      <td>01/12/2021</td>
      <td>VIOLENCIA FAMILIAR</td>
      <td>19.259511</td>
      <td>-99.101052</td>
    </tr>
    <tr>
      <th>780286</th>
      <td>19/07/2021</td>
      <td>ABUSO SEXUAL</td>
      <td>19.352580</td>
      <td>-99.019935</td>
    </tr>
    <tr>
      <th>780320</th>
      <td>12/12/2021</td>
      <td>ABUSO SEXUAL</td>
      <td>19.325796</td>
      <td>-99.069686</td>
    </tr>
  </tbody>
</table>
<p>31613 rows × 4 columns</p>
</div>
</div>


and continuing with the body. List all the values for 'Crime':


```python
vaw['Crime'].unique()
```




    array(['VIOLENCIA FAMILIAR', 'ABUSO SEXUAL', 'ACOSO SEXUAL',
           'CONTRA LA INTIMIDAD SEXUAL',
           'ACOSO SEXUAL AGRAVADO EN CONTRA DE MENORES', 'FEMINICIDIO',
           'FEMINICIDIO POR ARMA BLANCA', 'TENTATIVA DE FEMINICIDIO',
           'FEMINICIDIO POR DISPARO DE ARMA DE FUEGO',
           'PRIVACION DE LA LIBERTAD PERSONAL (REALIZAR ACTO SEXUAL)',
           'FEMINICIDIO POR GOLPES'], dtype=object)



Several types of 'FEMICIDE' are in. I use a simple regular expression to join them and replace them with the English word.


```python
vaw['Crime'] = vaw['Crime'].str.replace(r'.*FEMINICIDIO.*', r'FEMICIDE', regex = True)
```

Conclude the translation for the other crime types.


```python
from_text = ['VIOLENCIA FAMILIAR', 'ABUSO SEXUAL', 'ACOSO SEXUAL',
             'CONTRA LA INTIMIDAD SEXUAL', 'ACOSO SEXUAL AGRAVADO EN CONTRA DE MENORES',
             'FEMICIDE', 'PRIVACION DE LA LIBERTAD PERSONAL (REALIZAR ACTO SEXUAL)']

replace_with = ['DOMESTIC VIOLENCE', 'SEXUAL ABUSE', 'SEXUAL HARASSMENT',
        'AGAINST SEXUAL INTIMACY', 'AGGRAVATED SEXUAL HARASSMENT AGAINST MINORS', 'FEMICIDE',
        'DEPRIVATION OF PERSONAL LIBERTY (PERFORMING A SEXUAL ACT)']

vaw = vaw.replace(dict(zip(from_text, replace_with)))
```


```python
vaw.head()
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
      <th>Time</th>
      <th>Crime</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>480286</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.198153</td>
      <td>-99.026277</td>
    </tr>
    <tr>
      <th>480289</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.193027</td>
      <td>-99.095006</td>
    </tr>
    <tr>
      <th>480302</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.336963</td>
      <td>-99.152265</td>
    </tr>
    <tr>
      <th>480306</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.561525</td>
      <td>-99.146590</td>
    </tr>
    <tr>
      <th>480320</th>
      <td>01/01/2021</td>
      <td>DOMESTIC VIOLENCE</td>
      <td>19.395045</td>
      <td>-99.061060</td>
    </tr>
  </tbody>
</table>
</div>
</div>



```python
#save file
vaw.to_csv('data/vaw_filtered.csv', index = False)
```

# Neighbourhood

Add to neighbourhood variables the total population per unit by merging with 'Population per Neighbourhood'.

* Date source:

  [CDMX neighborhoods - Mexico City Open Data Portal](https://datos.cdmx.gob.mx/dataset/alcaldias/resource/8648431b-4f34-4f1a-a4b1-19142f944300)

* Data Structure:

  Structure of the data is described in a separate excel file [download here](https://datos.cdmx.gob.mx/dataset/alcaldias/resource/e4a9b05f-c480-45fb-a62c-6d4e39c5180e)


```python
import geopandas as gpd
```


```python
# Read the data
gdf = gpd.read_file('data/alcaldías_cdmx/alcaldias_cdmx.shp')
gdf = gdf.rename(columns = {'nomgeo':'NOMGEO'})
#gdf.head(5)
```

**Population per Neighbourhood**

* Date source:

  [CDMX Population per Neighbourhood - Mexico City Open Data Portal](https://datos.cdmx.gob.mx/dataset/poblacion-total-y-tasa-de-crecimiento-media-anual-1990-2020-por-alcaldia)


* Data Structure:

  Structure of the data is described in a separate excel file [download here](https://datos.cdmx.gob.mx/dataset/poblacion-total-y-tasa-de-crecimiento-media-anual-1990-2020-por-alcaldia/resource/ffa231fc-e0d2-4583-8a99-fd421fc9b442)


```python
df = pd.read_csv('data/poblacion_total_tasa_crecimiento_1.1.csv')
df = df[df['Año'] == 2020]
df = df.reset_index(drop=True)
#df
```


```python
# drop the last summary row
df = df.drop([16])
df = df.rename(columns = {'Alcaldia':'NOMGEO', 'Población total': 'POBTOT' })
df = df.apply(lambda x: x.astype(str).str.upper())
```


```python
merge = gdf.merge(df, on='NOMGEO')
merge = merge.rename(columns = {'cvegeo':'CVEGEO'})
merge['POBTOT'] = merge['POBTOT'].str.replace(',', '').astype(int)
```


```python
gdf = merge[['CVEGEO', 'NOMGEO', 'POBTOT', 'geometry']]
```


```python
# Write to GeoJSON

gdf.to_file("data/CDMX_municipalities.gjson", driver="GeoJSON")
```

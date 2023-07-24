---
layout: post
title: Predicting car prices with scikit-learn
tags:
  - Python
  - scikit-learn
---

In this notebook I will attempt to predict automobile prices using Python and its data analysis and machine learning packages such as `pandas` and `scikit-learn`. Forecasting car prices can be useful for businesses in general: insurance companies can use this information to calculate their premia; websites and enterprises can provide estimates even when asking prices are not available in a specific application; enterprises can set contracts where a car's resale value must be defined _a priori_ with greater information, or determine if a car is overvalued or undervalued with respect to the market.

My data source is [Mercado Livre](www.mercadolivre.com.br), a widely used e-commerce platform in Brazil. For simplicity, and also to isolate other geographical factors that affect prices, this research restricts to ads in the city of Porto Alegre/RS. I also excluded trucks and minivans from the analysis. While new cars can be announced, most of the cars advertised are used. To publish an ad, the seller must fill information about the car, such as brand, model, mileage, engine power and additional features. It is a common belief that this features can help predict a car's asking price in the market, and in our methodology I will explore them to improve our predictions (of course, there also will be unobservable factors that affect prices). A popular source for automobile prices in Brazil is the [FIPE table](http://www.veiculos.fipe.org.br/). In the table, average prices are calculated from newspaper and web ads. Data from the FIPE table can be improved here, as learning models can benefit from data updates, predict prices for cars with a specific set of features and location.

Along the notebook, I will go through all steps of data analysis, with code and commentary.

# Data wrangling

First, let's import the required packages for this step:


```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
sns.set()

import numpy as np
import json
import requests
```

To access data on ads, I'll make requests to the MercadoLivre's API. We must build the query URL, using category and location identifiers that can be found in the API [documentation site](http://developers.mercadolivre.com.br/).


```python
# parameters
category = 'MLB1744' # cars and trucks
city     = 'TUxCUFJJT0xkYzM0' # state of rs
limit    = '50'
offset   = [str(i) for i in list(range(50,10000,50))]

# access token
with open('ml_token.txt') as file:  
    token = file.read()

#url = 'https://api.mercadolibre.com/sites/MLB/search?category='+category+'&city='+city+'&limit='+limit+'&offset='+offset+'&access_token='+token
```

Users are only allowed to fetch 50 items per request - up to 10000 items - and this is controlled by the `offset` parameter. Using `requests`, we can download all data:


```python
responses = []
for off in offset:
    url = 'https://api.mercadolibre.com/sites/MLB/search?category='+category+'&state='+city+'&limit='+limit+'&offset='+off+'&access_token='+token
    responses.append(requests.get(url))

respd = [i.json() for i in responses]
```

The `.json` method transforms the data into a Python dictionary. Looking at the structure of data, I find that ad entries are stored under key `results`:


```python
respd[0].keys()
```




    dict_keys(['site_id', 'paging', 'results', 'secondary_results', 'related_results', 'sort', 'available_sorts', 'filters', 'available_filters'])




```python
# tests
data1 = [pd.io.json.json_normalize(i['results'], sep='_') for i in respd]
```

The result is a list of pandas DataFrames. We can append all datasets using `concat`:


```python
data1 = pd.concat(data1, sort=False).reset_index(drop=True)
data1.head()
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
      <th>accepts_mercadopago</th>
      <th>address_area_code</th>
      <th>address_city_id</th>
      <th>address_city_name</th>
      <th>address_phone1</th>
      <th>address_state_id</th>
      <th>address_state_name</th>
      <th>attributes</th>
      <th>available_quantity</th>
      <th>buying_mode</th>
      <th>...</th>
      <th>shipping_logistic_type</th>
      <th>shipping_mode</th>
      <th>shipping_store_pick_up</th>
      <th>shipping_tags</th>
      <th>site_id</th>
      <th>sold_quantity</th>
      <th>stop_time</th>
      <th>tags</th>
      <th>thumbnail</th>
      <th>title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>True</td>
      <td></td>
      <td>TUxCQ0NBTjcxNTM</td>
      <td>Canoas</td>
      <td></td>
      <td>TUxCUFJJT0xkYzM0</td>
      <td>Rio Grande Do Sul</td>
      <td>[{'value_id': '2230581', 'value_name': 'Usado'...</td>
      <td>1</td>
      <td>classified</td>
      <td>...</td>
      <td>None</td>
      <td>not_specified</td>
      <td>False</td>
      <td>[]</td>
      <td>MLB</td>
      <td>0</td>
      <td>2019-02-24T04:00:00.000Z</td>
      <td>[poor_quality_thumbnail, poor_quality_picture,...</td>
      <td>http://mlb-s1-p.mlstatic.com/757910-MLB2918517...</td>
      <td>Hyundai Hb20s 1.6 Comfort Plus 16v Flex 4p Manual</td>
    </tr>
    <tr>
      <th>1</th>
      <td>True</td>
      <td></td>
      <td>TUxCQ0VOQzZhYTdi</td>
      <td>Encantado</td>
      <td></td>
      <td>TUxCUFJJT0xkYzM0</td>
      <td>Rio Grande Do Sul</td>
      <td>[{'value_id': '2230581', 'value_name': 'Usado'...</td>
      <td>1</td>
      <td>classified</td>
      <td>...</td>
      <td>None</td>
      <td>not_specified</td>
      <td>False</td>
      <td>[]</td>
      <td>MLB</td>
      <td>0</td>
      <td>2019-02-21T22:16:47.000Z</td>
      <td>[poor_quality_picture, poor_quality_thumbnail,...</td>
      <td>http://mlb-s1-p.mlstatic.com/671101-MLB2907376...</td>
      <td>Chevrolet Agile Ltz - Fernando Multimarcas</td>
    </tr>
    <tr>
      <th>2</th>
      <td>True</td>
      <td></td>
      <td>TUxCQ1BPUjgwZTJl</td>
      <td>Porto Alegre</td>
      <td></td>
      <td>TUxCUFJJT0xkYzM0</td>
      <td>Rio Grande Do Sul</td>
      <td>[{'source': 1, 'id': 'ITEM_CONDITION', 'name':...</td>
      <td>1</td>
      <td>classified</td>
      <td>...</td>
      <td>None</td>
      <td>not_specified</td>
      <td>False</td>
      <td>[]</td>
      <td>MLB</td>
      <td>0</td>
      <td>2019-02-09T09:34:55.000Z</td>
      <td>[good_quality_picture, good_quality_thumbnail,...</td>
      <td>http://mlb-s2-p.mlstatic.com/962446-MLB2896708...</td>
      <td>Renault Grand Scenic 2002</td>
    </tr>
    <tr>
      <th>3</th>
      <td>True</td>
      <td></td>
      <td>TUxCQ0NBWDgxMzcw</td>
      <td>Caxias do Sul</td>
      <td></td>
      <td>TUxCUFJJT0xkYzM0</td>
      <td>Rio Grande Do Sul</td>
      <td>[{'id': 'ITEM_CONDITION', 'name': 'Condição do...</td>
      <td>1</td>
      <td>classified</td>
      <td>...</td>
      <td>None</td>
      <td>not_specified</td>
      <td>False</td>
      <td>[]</td>
      <td>MLB</td>
      <td>0</td>
      <td>2019-03-19T04:00:00.000Z</td>
      <td>[only_html_description, immediate_payment]</td>
      <td>http://mlb-s2-p.mlstatic.com/668212-MLB2919195...</td>
      <td>Hb20 1.6 Comfort Plus 16v Flex 4p</td>
    </tr>
    <tr>
      <th>4</th>
      <td>True</td>
      <td></td>
      <td>TUxCQ1BPUjgwZTJl</td>
      <td>Porto Alegre</td>
      <td></td>
      <td>TUxCUFJJT0xkYzM0</td>
      <td>Rio Grande Do Sul</td>
      <td>[{'value_name': 'Automática', 'value_struct': ...</td>
      <td>1</td>
      <td>classified</td>
      <td>...</td>
      <td>None</td>
      <td>not_specified</td>
      <td>False</td>
      <td>[]</td>
      <td>MLB</td>
      <td>0</td>
      <td>2019-02-16T04:00:00.000Z</td>
      <td>[dragged_visits, good_quality_picture, good_qu...</td>
      <td>http://mlb-s1-p.mlstatic.com/968792-MLB2777645...</td>
      <td>Volvo Xc90 2.0 T6 Inscription Drive-e 5p</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 73 columns</p>
</div>



In each entry, we have a series of characteristics on the car, such as make, model, asking price, features and location. Next, I will transform the data into a pandas DataFrame for further manipulation. This requires a few steps, as the data is structured in a list of dicts:


```python
cols = ['title','price','address_city_name','attributes', 'id',
       'location_latitude', 'location_longitude',
       'permalink',
       'seller_car_dealer']

data1 = data1.loc[:,cols]
data1.head()
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
      <th>title</th>
      <th>price</th>
      <th>address_city_name</th>
      <th>attributes</th>
      <th>id</th>
      <th>location_latitude</th>
      <th>location_longitude</th>
      <th>permalink</th>
      <th>seller_car_dealer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hyundai Hb20s 1.6 Comfort Plus 16v Flex 4p Manual</td>
      <td>46990.0</td>
      <td>Canoas</td>
      <td>[{'value_id': '2230581', 'value_name': 'Usado'...</td>
      <td>MLB1167035642</td>
      <td>-29.9188</td>
      <td>-51.1585</td>
      <td>https://carro.mercadolivre.com.br/MLB-11670356...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chevrolet Agile Ltz - Fernando Multimarcas</td>
      <td>28800.0</td>
      <td>Encantado</td>
      <td>[{'value_id': '2230581', 'value_name': 'Usado'...</td>
      <td>MLB1159937620</td>
      <td>-29.2249</td>
      <td>-51.8895</td>
      <td>https://carro.mercadolivre.com.br/MLB-11599376...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Renault Grand Scenic 2002</td>
      <td>8500.0</td>
      <td>Porto Alegre</td>
      <td>[{'source': 1, 'id': 'ITEM_CONDITION', 'name':...</td>
      <td>MLB1152825359</td>
      <td>-30.0346</td>
      <td>-51.2177</td>
      <td>https://carro.mercadolivre.com.br/MLB-11528253...</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hb20 1.6 Comfort Plus 16v Flex 4p</td>
      <td>44890.0</td>
      <td>Caxias do Sul</td>
      <td>[{'id': 'ITEM_CONDITION', 'name': 'Condição do...</td>
      <td>MLB1167546961</td>
      <td>-29.1634</td>
      <td>-51.1797</td>
      <td>https://carro.mercadolivre.com.br/MLB-11675469...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Volvo Xc90 2.0 T6 Inscription Drive-e 5p</td>
      <td>339950.0</td>
      <td>Porto Alegre</td>
      <td>[{'value_name': 'Automática', 'value_struct': ...</td>
      <td>MLB1165970712</td>
      <td>-30.0554</td>
      <td>-51.2224</td>
      <td>https://carro.mercadolivre.com.br/MLB-11659707...</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



It's a great step forward, however information on a car's attributes are still trapped in a list of dicts. Hence any attempts at using the previous `json_serialize` method will fail. As an example, let's apply the function to the feature alone:


```python
pd.io.json.json_normalize(data1.attributes[0], sep='_').head()
```

Now, I will reshape the data in order to have a single row per advertising. Let's define a helper function:


```python
def reshape_(data):
    return data.loc[:,['id','value_name']].set_index('id').transpose()
```

We must now loop over samples and then join all data. _For_ loops can be avoided, using Python's list comprehensions:


```python
df_temp = [reshape_(pd.io.json.json_normalize(i)) for i in data1.attributes]

data2 = pd.concat(df_temp, sort=False).reset_index(drop=True)
del df_temp
data2.tail()
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
      <th>ITEM_CONDITION</th>
      <th>TRANSMISSION</th>
      <th>ENGINE_DISPLACEMENT</th>
      <th>BRAND</th>
      <th>DOORS</th>
      <th>FUEL_TYPE</th>
      <th>KILOMETERS</th>
      <th>MODEL</th>
      <th>TRIM</th>
      <th>VEHICLE_YEAR</th>
      <th>TRACTION_CONTROL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9943</th>
      <td>Usado</td>
      <td>Manual</td>
      <td>999 cc</td>
      <td>Volkswagen</td>
      <td>5</td>
      <td>Gasolina e álcool</td>
      <td>100000 km</td>
      <td>Fox</td>
      <td>1.0 Vht Trend Total Flex 5p</td>
      <td>2009</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>9944</th>
      <td>Usado</td>
      <td>Manual</td>
      <td>999 cc</td>
      <td>Ford</td>
      <td>5</td>
      <td>Gasolina</td>
      <td>130000 km</td>
      <td>Fiesta</td>
      <td>1.0 Street 5p</td>
      <td>2004</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>9945</th>
      <td>Usado</td>
      <td>Manual</td>
      <td>1360 cc</td>
      <td>Peugeot</td>
      <td>5</td>
      <td>Gasolina e álcool</td>
      <td>124 km</td>
      <td>206</td>
      <td>1.4 Presence Flex 5p</td>
      <td>2008</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>9946</th>
      <td>Usado</td>
      <td>Manual</td>
      <td>999 cc</td>
      <td>Fiat</td>
      <td>5</td>
      <td>Gasolina e álcool</td>
      <td>95800 km</td>
      <td>Palio</td>
      <td>1.0 Fire Celebration Flex 5p</td>
      <td>2009</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>9947</th>
      <td>Usado</td>
      <td>Manual</td>
      <td>1598 cc</td>
      <td>Ford</td>
      <td>5</td>
      <td>Gasolina</td>
      <td>190000 km</td>
      <td>Focus</td>
      <td>1.6 Gl 5p</td>
      <td>2007</td>
      <td>Dianteira</td>
    </tr>
  </tbody>
</table>
</div>



Finally, we'll merge with the original data:


```python
df = pd.concat([data1,data2], axis=1)
```


```python
df.head()
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
      <th>price</th>
      <th>address_city_name</th>
      <th>location_latitude</th>
      <th>location_longitude</th>
      <th>seller_car_dealer</th>
      <th>transmission</th>
      <th>engine_displacement</th>
      <th>brand</th>
      <th>doors</th>
      <th>fuel_type</th>
      <th>kilometers</th>
      <th>model</th>
      <th>vehicle_year</th>
      <th>traction_control</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>46990.0</td>
      <td>Canoas</td>
      <td>-29.918818</td>
      <td>-51.158540</td>
      <td>True</td>
      <td>Manual</td>
      <td>1591.0</td>
      <td>hyundai</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>72.258</td>
      <td>hb20s</td>
      <td>2017.0</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>1</th>
      <td>28800.0</td>
      <td>Encantado</td>
      <td>-29.224850</td>
      <td>-51.889533</td>
      <td>True</td>
      <td>Manual</td>
      <td>1389.0</td>
      <td>chevrolet</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>74.000</td>
      <td>agile</td>
      <td>2011.0</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>3</th>
      <td>44890.0</td>
      <td>Caxias do Sul</td>
      <td>-29.163403</td>
      <td>-51.179668</td>
      <td>True</td>
      <td>Manual</td>
      <td>1591.0</td>
      <td>hyundai</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>39.293</td>
      <td>hb20</td>
      <td>2018.0</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>5</th>
      <td>52900.0</td>
      <td>Santa Maria</td>
      <td>-29.696541</td>
      <td>-53.799310</td>
      <td>True</td>
      <td>Automática</td>
      <td>1598.0</td>
      <td>volkswagen</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>30.600</td>
      <td>crossfox</td>
      <td>2015.0</td>
      <td>4x2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>61900.0</td>
      <td>Santa Maria</td>
      <td>-29.696541</td>
      <td>-53.799310</td>
      <td>True</td>
      <td>Automática</td>
      <td>999.0</td>
      <td>ford</td>
      <td>5.0</td>
      <td>Gasolina</td>
      <td>9.345</td>
      <td>fiesta</td>
      <td>2017.0</td>
      <td>4x2</td>
    </tr>
  </tbody>
</table>
</div>



Some more data cleaning, to deal with strings and data types:


```python
df.drop(['TRIM','ITEM_CONDITION','title','attributes','id','permalink'], axis=1, inplace=True)
df.columns = map(str.lower, df.columns)

#remove unwanted strings
cols = ['location_latitude', 'location_longitude','engine_displacement','kilometers','vehicle_year','doors']
df.loc[:,cols] = df.loc[:,cols].replace(regex=True,to_replace=r'\D',value=r'')

# replace empty strings with NaN
df.replace({'':np.nan}, inplace=True)

# set brand and model strings to lowercase
df.brand = df.brand.str.lower()
df.model = df.model.str.lower()
```


```python
# type conversion
df = df.astype({'location_latitude':float, 'location_longitude':float,'engine_displacement':float,
                'kilometers':float,'vehicle_year':float,'doors':float})

df.dtypes
```




    price                  float64
    address_city_name       object
    location_latitude      float64
    location_longitude     float64
    seller_car_dealer         bool
    transmission            object
    engine_displacement    float64
    brand                   object
    doors                  float64
    fuel_type               object
    kilometers             float64
    model                   object
    vehicle_year           float64
    traction_control        object
    dtype: object



Now, I will attempt to detect outliers in numerical features. We must note that some extreme values are not outliers (i.e. we will have feature `kilometers` set to zero for new cars). Outlier detection will generate missing values that must be treated later.


```python
df.describe().round(2)
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
      <th>price</th>
      <th>location_latitude</th>
      <th>location_longitude</th>
      <th>engine_displacement</th>
      <th>doors</th>
      <th>kilometers</th>
      <th>vehicle_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>9740.00</td>
      <td>9740.00</td>
      <td>9740.00</td>
      <td>9740.00</td>
      <td>9740.00</td>
      <td>9740.00</td>
      <td>9740.00</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>47753.99</td>
      <td>-29.70</td>
      <td>-51.29</td>
      <td>1653.57</td>
      <td>3.97</td>
      <td>57.63</td>
      <td>2012.58</td>
    </tr>
    <tr>
      <th>std</th>
      <td>32267.50</td>
      <td>1.17</td>
      <td>0.99</td>
      <td>528.68</td>
      <td>0.76</td>
      <td>51.19</td>
      <td>4.39</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8999.00</td>
      <td>-33.69</td>
      <td>-57.55</td>
      <td>994.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>1960.00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>27900.00</td>
      <td>-30.02</td>
      <td>-51.20</td>
      <td>1368.00</td>
      <td>4.00</td>
      <td>21.00</td>
      <td>2011.00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>37900.00</td>
      <td>-29.95</td>
      <td>-51.18</td>
      <td>1598.00</td>
      <td>4.00</td>
      <td>54.00</td>
      <td>2013.00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>56040.00</td>
      <td>-29.70</td>
      <td>-51.14</td>
      <td>1975.00</td>
      <td>4.00</td>
      <td>83.00</td>
      <td>2015.00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>249000.00</td>
      <td>-6.93</td>
      <td>-37.88</td>
      <td>3960.00</td>
      <td>7.00</td>
      <td>920.00</td>
      <td>2019.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# fix extreme values
df.price.replace({df.price.max(): np.nan}, inplace=True)
df.engine_displacement[df.engine_displacement < 900] = np.nan
df.engine_displacement[df.engine_displacement > 4000] = np.nan
df.kilometers.replace({df.kilometers.max(): np.nan}, inplace=True)
df.vehicle_year.replace({df.vehicle_year.min(): np.nan}, inplace=True)
```

Finally, we must deal with missing values, since `scikit-learn` does not accept them. It is often more fruitful to fill in missing values, rather than dropping whole samples.


```python
# find missing values
df.isna().sum()
```




    price                  0
    address_city_name      0
    location_latitude      0
    location_longitude     0
    seller_car_dealer      0
    transmission           0
    engine_displacement    0
    brand                  0
    doors                  0
    fuel_type              0
    kilometers             0
    model                  0
    vehicle_year           0
    traction_control       0
    dtype: int64




```python
df.transmission       = df.transmission.fillna('Manual')
df.kilometers         = df.kilometers.fillna(df.kilometers.mean())
df.price              = df.price.fillna(df.price.mean())
df.traction_control   = df.traction_control.fillna('Dianteira')
df.location_latitude  = df.location_latitude.fillna(method='ffill')
df.location_longitude = df.location_longitude.fillna(method='ffill')
df.vehicle_year       = df.vehicle_year.fillna(df.vehicle_year.mean())
```

Features `engine_displacement` still exhibit lots of missing values. For `engine_displacement`, my strategy was to fill with the mode (or median) values of features according to each car model. This seems to be a better approach than filling with the average engine size of all cars.


```python
import statistics
def mode_or_median(var):
    try:
        return statistics.mode(var)
    except statistics.StatisticsError: # there may be no unique value
        return np.median(var)
```


```python
avg_eng = df.groupby('model')['engine_displacement'].apply(mode_or_median)
avg_eng.fillna(avg_eng.median(),inplace=True) # fill the remaining NaN's
avg_eng.sample(10)
```

Now, we will insert the average values only when data were initially missing:


```python
df.set_index('model', drop=False, inplace=True)
df.update(avg_eng, overwrite=False) # overwrite=False only updates NaN's
df.reset_index(drop=True, inplace=True)
```

Last, I will drop extreme values from our target distribution of car prices. First, because luxury car prices are harder to predict (because of fewer data points), and also because some entries are just wrong (i.e. ads for older cars whose prices have been mistyped).


```python
# keep values within interval
df = df[(df.price > df.price.quantile(0.01)) & (df.price < df.price.quantile(0.99))]
```

Finally, we have a dataset ready for analysis:


```python
df.head()
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
      <th>price</th>
      <th>address_city_name</th>
      <th>location_latitude</th>
      <th>location_longitude</th>
      <th>seller_car_dealer</th>
      <th>transmission</th>
      <th>engine_displacement</th>
      <th>brand</th>
      <th>doors</th>
      <th>fuel_type</th>
      <th>kilometers</th>
      <th>model</th>
      <th>vehicle_year</th>
      <th>traction_control</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>46990.0</td>
      <td>Canoas</td>
      <td>-29.918818</td>
      <td>-51.158540</td>
      <td>True</td>
      <td>Manual</td>
      <td>1591.0</td>
      <td>hyundai</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>72.258</td>
      <td>hb20s</td>
      <td>2017.0</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>1</th>
      <td>28800.0</td>
      <td>Encantado</td>
      <td>-29.224850</td>
      <td>-51.889533</td>
      <td>True</td>
      <td>Manual</td>
      <td>1389.0</td>
      <td>chevrolet</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>74.000</td>
      <td>agile</td>
      <td>2011.0</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>3</th>
      <td>44890.0</td>
      <td>Caxias do Sul</td>
      <td>-29.163403</td>
      <td>-51.179668</td>
      <td>True</td>
      <td>Manual</td>
      <td>1591.0</td>
      <td>hyundai</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>39.293</td>
      <td>hb20</td>
      <td>2018.0</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>5</th>
      <td>52900.0</td>
      <td>Santa Maria</td>
      <td>-29.696541</td>
      <td>-53.799310</td>
      <td>True</td>
      <td>Automática</td>
      <td>1598.0</td>
      <td>volkswagen</td>
      <td>4.0</td>
      <td>Gasolina e álcool</td>
      <td>30.600</td>
      <td>crossfox</td>
      <td>2015.0</td>
      <td>4x2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>61900.0</td>
      <td>Santa Maria</td>
      <td>-29.696541</td>
      <td>-53.799310</td>
      <td>True</td>
      <td>Automática</td>
      <td>999.0</td>
      <td>ford</td>
      <td>5.0</td>
      <td>Gasolina</td>
      <td>9.345</td>
      <td>fiesta</td>
      <td>2017.0</td>
      <td>4x2</td>
    </tr>
  </tbody>
</table>
</div>



# Exploratory data analysis

First, let's check descriptive statistics:


```python
#df.to_csv('~/mlcars.csv', index=False)
df = pd.read_csv('C:/Users/gsalt/mlcars.csv')
```


```python
df.describe(include='all').round(2)
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
      <th>price</th>
      <th>address_city_name</th>
      <th>location_latitude</th>
      <th>location_longitude</th>
      <th>seller_car_dealer</th>
      <th>transmission</th>
      <th>engine_displacement</th>
      <th>brand</th>
      <th>doors</th>
      <th>fuel_type</th>
      <th>kilometers</th>
      <th>model</th>
      <th>vehicle_year</th>
      <th>traction_control</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>9740.00</td>
      <td>9740</td>
      <td>9740.00</td>
      <td>9740.00</td>
      <td>9740</td>
      <td>9740</td>
      <td>9740.00</td>
      <td>9740</td>
      <td>9740.00</td>
      <td>9740</td>
      <td>9740.00</td>
      <td>9740</td>
      <td>9740.00</td>
      <td>9740</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>NaN</td>
      <td>147</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>4</td>
      <td>NaN</td>
      <td>49</td>
      <td>NaN</td>
      <td>12</td>
      <td>NaN</td>
      <td>440</td>
      <td>NaN</td>
      <td>5</td>
    </tr>
    <tr>
      <th>top</th>
      <td>NaN</td>
      <td>Porto Alegre</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>True</td>
      <td>Manual</td>
      <td>NaN</td>
      <td>chevrolet</td>
      <td>NaN</td>
      <td>Gasolina e álcool</td>
      <td>NaN</td>
      <td>onix</td>
      <td>NaN</td>
      <td>Dianteira</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>NaN</td>
      <td>4103</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9019</td>
      <td>6008</td>
      <td>NaN</td>
      <td>1514</td>
      <td>NaN</td>
      <td>6910</td>
      <td>NaN</td>
      <td>280</td>
      <td>NaN</td>
      <td>7812</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>47753.99</td>
      <td>NaN</td>
      <td>-29.70</td>
      <td>-51.29</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1653.57</td>
      <td>NaN</td>
      <td>3.97</td>
      <td>NaN</td>
      <td>57.63</td>
      <td>NaN</td>
      <td>2012.58</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>std</th>
      <td>32267.50</td>
      <td>NaN</td>
      <td>1.17</td>
      <td>0.99</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>528.68</td>
      <td>NaN</td>
      <td>0.76</td>
      <td>NaN</td>
      <td>51.19</td>
      <td>NaN</td>
      <td>4.39</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8999.00</td>
      <td>NaN</td>
      <td>-33.69</td>
      <td>-57.55</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>994.00</td>
      <td>NaN</td>
      <td>0.00</td>
      <td>NaN</td>
      <td>0.00</td>
      <td>NaN</td>
      <td>1960.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>27900.00</td>
      <td>NaN</td>
      <td>-30.02</td>
      <td>-51.20</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1368.00</td>
      <td>NaN</td>
      <td>4.00</td>
      <td>NaN</td>
      <td>21.00</td>
      <td>NaN</td>
      <td>2011.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>37900.00</td>
      <td>NaN</td>
      <td>-29.95</td>
      <td>-51.18</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1598.00</td>
      <td>NaN</td>
      <td>4.00</td>
      <td>NaN</td>
      <td>54.00</td>
      <td>NaN</td>
      <td>2013.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>56040.00</td>
      <td>NaN</td>
      <td>-29.70</td>
      <td>-51.14</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1975.00</td>
      <td>NaN</td>
      <td>4.00</td>
      <td>NaN</td>
      <td>83.00</td>
      <td>NaN</td>
      <td>2015.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>max</th>
      <td>249000.00</td>
      <td>NaN</td>
      <td>-6.93</td>
      <td>-37.88</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3960.00</td>
      <td>NaN</td>
      <td>7.00</td>
      <td>NaN</td>
      <td>920.00</td>
      <td>NaN</td>
      <td>2019.00</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



The `describe` method is very useful to find strange values left in data, such as found in _engine displacement_ and _kilometers_.

The correlation matrix:


```python
sns.heatmap(df.corr().round(2), annot=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x285fae7c208>




![png](/assets/images/mlcars/output_45_1.png)


The most popular cars announced and the average price:


```python
df.groupby(['brand','model']).agg({'price':['mean','count']}).sort_values(('price','count'), ascending=False)[:10]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th colspan="2" halign="left">price</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th>mean</th>
      <th>count</th>
    </tr>
    <tr>
      <th>brand</th>
      <th>model</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>chevrolet</th>
      <th>onix</th>
      <td>40334.564250</td>
      <td>280</td>
    </tr>
    <tr>
      <th>fiat</th>
      <th>palio</th>
      <td>25952.777778</td>
      <td>270</td>
    </tr>
    <tr>
      <th>renault</th>
      <th>sandero</th>
      <td>32828.739837</td>
      <td>246</td>
    </tr>
    <tr>
      <th>volkswagen</th>
      <th>gol</th>
      <td>23750.466667</td>
      <td>240</td>
    </tr>
    <tr>
      <th>ford</th>
      <th>fiesta</th>
      <td>32778.090909</td>
      <td>231</td>
    </tr>
    <tr>
      <th>volkswagen</th>
      <th>fox</th>
      <td>31756.559633</td>
      <td>218</td>
    </tr>
    <tr>
      <th>fiat</th>
      <th>uno</th>
      <td>27420.529126</td>
      <td>206</td>
    </tr>
    <tr>
      <th>ford</th>
      <th>ecosport</th>
      <td>48480.684211</td>
      <td>190</td>
    </tr>
    <tr>
      <th>citroën</th>
      <th>c3</th>
      <td>32972.803371</td>
      <td>178</td>
    </tr>
    <tr>
      <th>ford</th>
      <th>ka</th>
      <td>32543.722543</td>
      <td>173</td>
    </tr>
  </tbody>
</table>
</div>



The same query as before, now considering the manufacturing year of the cars:


```python
df.groupby(['brand','model','vehicle_year']).agg({'price':['mean','count']}).sort_values(('price','count'), ascending=False)[:10]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th colspan="2" halign="left">price</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th>mean</th>
      <th>count</th>
    </tr>
    <tr>
      <th>brand</th>
      <th>model</th>
      <th>vehicle_year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fiat</th>
      <th>mobi</th>
      <th>2018.0</th>
      <td>33558.644068</td>
      <td>59</td>
    </tr>
    <tr>
      <th>ford</th>
      <th>ka</th>
      <th>2018.0</th>
      <td>39714.827586</td>
      <td>58</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">chevrolet</th>
      <th rowspan="2" valign="top">onix</th>
      <th>2015.0</th>
      <td>39301.210526</td>
      <td>57</td>
    </tr>
    <tr>
      <th>2018.0</th>
      <td>39498.392857</td>
      <td>56</td>
    </tr>
    <tr>
      <th>ford</th>
      <th>fiesta</th>
      <th>2014.0</th>
      <td>33070.696429</td>
      <td>56</td>
    </tr>
    <tr>
      <th>jeep</th>
      <th>renegade</th>
      <th>2016.0</th>
      <td>75305.489796</td>
      <td>49</td>
    </tr>
    <tr>
      <th>chevrolet</th>
      <th>onix</th>
      <th>2016.0</th>
      <td>42870.638085</td>
      <td>47</td>
    </tr>
    <tr>
      <th>fiat</th>
      <th>uno</th>
      <th>2018.0</th>
      <td>32799.565217</td>
      <td>46</td>
    </tr>
    <tr>
      <th>chevrolet</th>
      <th>onix</th>
      <th>2017.0</th>
      <td>43780.888889</td>
      <td>45</td>
    </tr>
    <tr>
      <th>renault</th>
      <th>sandero</th>
      <th>2018.0</th>
      <td>37992.222222</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>



Here are the most expensive brands, by average price:


```python
df.pivot_table(index=['brand'], values=['price'], aggfunc=np.mean).round(2).sort_values('price', ascending=False)[:10].plot.bar()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x285fb184828>




![png](/assets/images/mlcars/output_51_1.png)


How cars are located geographically:


```python
import cartopy.crs as ccrs
from cartopy.io import shapereader

kw = dict(resolution='50m', category='cultural',
          name='admin_1_states_provinces')

states_shp = shapereader.natural_earth(**kw)
shp = shapereader.Reader(states_shp)

subplot_kw = dict(projection=ccrs.PlateCarree())

fig, ax = plt.subplots(figsize=(7, 11),
                       subplot_kw=subplot_kw)
ax.set_extent([-57.5,-49.5, -34,-27])
ax.add_geometries(shp.geometries(), ccrs.PlateCarree(), facecolor='lightgreen')
ax.scatter(df.location_longitude, df.location_latitude, marker='.', c='green', zorder=2)
```




    <matplotlib.collections.PathCollection at 0x285869b1f98>




![png](/assets/images/mlcars/output_53_1.png)


The relationship between continuous features are key in our dataset. Are they linear? This is important as most models assume a linear relationship between features and the target.


```python
sns.pairplot(df[['price','kilometers','engine_displacement','vehicle_year']], diag_kind='kde')
```

    C:\Users\gsalt\Anaconda3\lib\site-packages\scipy\stats\stats.py:1713: FutureWarning: Using a non-tuple sequence for multidimensional indexing is deprecated; use `arr[tuple(seq)]` instead of `arr[seq]`. In the future this will be interpreted as an array index, `arr[np.array(seq)]`, which will result either in an error or a different result.
      return np.add.reduce(sorted[indexer] * weights, axis=axis) / sumval
    




    <seaborn.axisgrid.PairGrid at 0x285885ec6d8>




![png](/assets/images/mlcars/output_55_2.png)


# Model tests

First, let's identify the scope of our problem. Our target, `price`, is continuous, so we're dealing with a regression problem. In this section I will experiment with different regression techniques. But first, the dataset must be further processed: our categorical features must be translated into numeric. This can be done in pandas with `get_dummies` (or in scikit-learn with `OneHotEncoder`): it transforms a categorical feature into dummy variables that indicate which category an observation belongs.'

My theoretical model is currently specified as (simplified):

$$price_i = model_i + year_i + features_i + error_i$$

In a regression context, this means that all auto models depreciate at a rate determined by the coefficient associated with $$year$$. However, one would expect each model to lose value at a different rate (i.e. luxury, fuel-efficient or low-maintenance cars would preserve their value for longer). We can implement this by interacting features $$model$$ and $$year$$, that is, $$price_i = modelyear_i + features_i + error_i$$. The downside of this approach is that it may lead to overfitting.

Let's load the `scikit-learn` methods:


```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline, FeatureUnion
from sklearn.preprocessing import StandardScaler, OneHotEncoder, FunctionTransformer
from sklearn.model_selection import train_test_split
from sklearn import metrics

# disable standard scaler data conversion warning
import warnings
from sklearn.exceptions import DataConversionWarning
warnings.filterwarnings(action='ignore', category=DataConversionWarning)

X = df.drop(['price','address_city_name'], axis=1).copy()
y = df.price.values.copy() # prices in R$
```

Next, I split the data in both training and data sets. Training data will be used to fit the model, and the test set to assess model accuracy.


```python
# generate train and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)
```

In addition, we can perform standardization (centering of data around the average, also called scaling) of our numerical features using `StandardScaler`. Standardization can improve the accuracy of most models, on the other hand we lose interpretation of model coefficients such as the ones given by linear regression. Fortunately, we can use the `predict` method to give us model predictions. This step will be done inside a pipeline, after training and test data splits, to avoid [data leakage](https://machinelearningmastery.com/data-leakage-machine-learning/).


```python
# categorical features
cat_ft = ['seller_car_dealer', 'transmission', 'brand', 'fuel_type', 'model', 'traction_control']

# run once on full dataset to get all category values
temp = ColumnTransformer([('cat', OneHotEncoder(), cat_ft)]).fit(X)
cats = temp.named_transformers_['cat'].categories_

cat_tr = OneHotEncoder(categories=cats, sparse=False)

# numerical features
num_ft = ['location_latitude', 'location_longitude', 'engine_displacement', 'doors', 'kilometers', 'vehicle_year']
num_tr = StandardScaler()
#num_tr = FunctionTransformer(func=None)

# data transformer
data_tr = ColumnTransformer([('num', num_tr, num_ft), ('cat', cat_tr, cat_ft)])
```

## Model comparison

There are a lot of techniques that can be used in a regression problem. As a starting point, I will compare some of them in terms of scores and sum of errors:


```python
from sklearn import linear_model
from sklearn.neighbors import KNeighborsRegressor
from sklearn.dummy import DummyRegressor
from sklearn import ensemble

models = [DummyRegressor(),
          KNeighborsRegressor(),
          #linear_model.LinearRegression(), # ommited because of negative scores
          linear_model.Lasso(),
          linear_model.Ridge(),
          linear_model.ElasticNet(),
          ensemble.GradientBoostingRegressor(),
          ensemble.RandomForestRegressor(),
          ensemble.ExtraTreesRegressor()]

models_names = ['Dummy','K-nn','Lasso','Ridge','Elastic','Boost','Forest','Extra']
```


```python
scores = []
mse = []
mae = []

for model in models:
    pipe = Pipeline([
    ('features', data_tr),
    ('model', model)
    ])
    fits = pipe.fit(X_train,y_train)
    scores.append(metrics.r2_score(y_test, fits.predict(X_test)))
    mse.append(metrics.mean_squared_error(y_test, fits.predict(X_test)))
    mae.append(metrics.median_absolute_error(y_test, fits.predict(X_test)))
```

    C:\Users\gsalt\Anaconda3\lib\site-packages\sklearn\linear_model\coordinate_descent.py:492: ConvergenceWarning: Objective did not converge. You might want to increase the number of iterations. Fitting data with very small alpha may cause precision problems.
      ConvergenceWarning)
    C:\Users\gsalt\Anaconda3\lib\site-packages\sklearn\ensemble\forest.py:246: FutureWarning: The default value of n_estimators will change from 10 in version 0.20 to 100 in 0.22.
      "10 in version 0.20 to 100 in 0.22.", FutureWarning)
    C:\Users\gsalt\Anaconda3\lib\site-packages\sklearn\ensemble\forest.py:246: FutureWarning: The default value of n_estimators will change from 10 in version 0.20 to 100 in 0.22.
      "10 in version 0.20 to 100 in 0.22.", FutureWarning)
    

Let's take a look at the model $$R^2$$ scores (more is better), mean squared error and median absolute error (less is better):


```python
f, (ax1, ax2, ax3) = plt.subplots(ncols=3, sharex=True, sharey=False, figsize=(18,6))
ax1.bar(models_names, scores)
ax1.set_ylabel('$R^2$')
ax2.bar(models_names, mse)
ax2.set_ylabel('Mean squared error')
ax3.bar(models_names, mae)
ax3.set_ylabel('Median absolute error')
```




    Text(0, 0.5, 'Median absolute error')




![png](/assets/images/mlcars/output_66_1.png)


All model scores are calculated using regression residuals, so they are correlated. The $$R^2$$ score in most models performed at around 0.8, except the elastic net model, which peaked at 0.5 in the training set. Gradient boosting, random forest and extra trees had the best scores. However, $$R^2$$ scores should be taken with precaution as they are biased towards larger models. Mean squared error (MSE) is a metric that penalizes larger prediction errors. Last are the median absolute error (MAE). Absolute error measures are interesting because they inform average deviations in terms of our target (predicted car prices, in our case).

I'll stick with the random forest regressor, as it was less prone to overfitting in previous tests.

## Predictions

Now, we fit the chosen model and extract some predictions:


```python
mod = Pipeline([
    ('features', data_tr),
    ('model', ensemble.RandomForestRegressor(n_estimators=100, min_samples_leaf=1))])

mod_fit = mod.fit(X_train, y_train)

mod_fit.score(X_test,y_test)
```

Some example predictions from the model:


```python
pred = pd.DataFrame.from_dict({'predicted':mod_fit.predict(X_test), 'true':y_test})
pred['difference'] = pred.predicted - pred.true
pred.sample(n=10).round(2)
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
      <th>predicted</th>
      <th>true</th>
      <th>difference</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2386</th>
      <td>36963.40</td>
      <td>34990.0</td>
      <td>1973.40</td>
    </tr>
    <tr>
      <th>144</th>
      <td>162293.38</td>
      <td>150000.0</td>
      <td>12293.38</td>
    </tr>
    <tr>
      <th>1175</th>
      <td>31838.20</td>
      <td>32960.0</td>
      <td>-1121.80</td>
    </tr>
    <tr>
      <th>2155</th>
      <td>29880.00</td>
      <td>29900.0</td>
      <td>-20.00</td>
    </tr>
    <tr>
      <th>1052</th>
      <td>44718.00</td>
      <td>44500.0</td>
      <td>218.00</td>
    </tr>
    <tr>
      <th>1759</th>
      <td>32580.00</td>
      <td>31900.0</td>
      <td>680.00</td>
    </tr>
    <tr>
      <th>622</th>
      <td>27303.20</td>
      <td>23900.0</td>
      <td>3403.20</td>
    </tr>
    <tr>
      <th>2363</th>
      <td>40363.60</td>
      <td>42900.0</td>
      <td>-2536.40</td>
    </tr>
    <tr>
      <th>429</th>
      <td>32632.40</td>
      <td>33900.0</td>
      <td>-1267.60</td>
    </tr>
    <tr>
      <th>2709</th>
      <td>39779.80</td>
      <td>33900.0</td>
      <td>5879.80</td>
    </tr>
  </tbody>
</table>
</div>



The average prediction error in car prices is around -R$600. The value being negative means that the model tends to underestimate prices in general. The not-so-good news is that the standard deviation of predictions is pretty high.


```python
pred.difference.describe()
```




    count      2922.000000
    mean       -632.707264
    std       13053.008512
    min     -189274.200000
    25%       -2489.150000
    50%         150.560000
    75%        2949.950000
    max       80052.000000
    Name: difference, dtype: float64



## Model tuning

The random forest regressor have a few hyperparameters that can be tuned to improve overall accuracy. I will play around with `n_estimators` (the number of estimators/trees) and `min_samples_leaf` (the minimum number of samples to be in a tree branch). One might be tempted to test all parameters, but the computational costs quickly grow according to the number of parameters, possible values and cross-validation folds:


```python
from sklearn.model_selection import GridSearchCV

# cross validation folds
folds=3

# set parameter range for grid search
params = {'model__n_estimators': [10, 20, 30, 50, 100], 'model__min_samples_leaf': [1,2]}

grids = GridSearchCV(mod, param_grid=params, cv=folds)
grids_fit = grids.fit(X_train, y_train)

print('Best choice of parameter:', grids_fit.best_params_)
```

    Best choice of parameter: {'model__min_samples_leaf': 1, 'model__n_estimators': 100}
    

# Validation

First, I will perform a three-fold cross validation (that is, calculate training and test scores for three random sub-samples of data) and take the mean of scores:


```python
from sklearn.model_selection import cross_validate

mod_cross_val = cross_validate(mod, X, y=y, cv=folds, return_train_score=True)

print('Average train score:', mod_cross_val['train_score'].mean().round(4))
print('Average test score:',  mod_cross_val['test_score'].mean().round(4))
```

    Average train score: 0.9766709555336993
    Average test score: 0.8178101080682266
    

Next, the learning curve tells us how model prediction improves as we add training samples (that's when the machine is _learning_):


```python
from sklearn.model_selection import learning_curve

learn_tr_size, learn_train_sc, learn_test_sc = learning_curve(mod, X_train, y_train, cv=folds)

# calculate mean over cross-validation folds
learn_train_m = np.apply_along_axis(np.mean, 1, learn_train_sc)
learn_test_m  = np.apply_along_axis(np.mean, 1, learn_test_sc)

plt.plot(learn_tr_size, learn_train_m)
plt.plot(learn_tr_size, learn_test_m)
plt.title('Learning curve')
plt.xlabel('Training samples')
plt.ylabel('Scores')
plt.legend(['Train score','Test score'])
```




    <matplotlib.legend.Legend at 0x2859d6e8d68>




![png](/assets/images/mlcars/output_79_1.png)


The learning curve tells us that our model performs much better in training than in test data, although both scores improve as we add samples. Intuitively, this means that the model struggles to generalize to new, unknown cases. Improvements can be made in the following senses:

- *Model tuning*: Results from the learning curve are a sign of overfitting. A solution could be fitting a more parsimonious tree.
- *Feature engineering*: in a regression context, having a single coefficient for `kilometers` means that all cars depreciate at a constant rate. But we know that luxury or fuel-efficient models keep their value for longer. A solution would be the creation of features that are interactions of `model` and `vehicle_year`.


Observed Trend 1 : As the latitude increase, temperature decrease  

Observed Trend 2 : cloudiness donest get influenced by latitude
    
Observed Trend 3 : latitude increase, wind speed tend to decrease


```python
import json
from random import random
import math
import requests
from pprint import pprint
import time
import pandas as pd
import matplotlib.pyplot as plt 
from citipy import citipy
import openweathermapy.core as owm
from Config_WeatherAPI import api_key

```


```python
# Function to produce randome coordinate accoriding to arguments(n: number of result, c : center corrdinate, r: radius)
def rand_cluster(n,c,r):
    """returns n random points in disk of radius r centered at c"""
    x,y = c
    points = []
    for i in range(n):
        theta = 2*math.pi*random()
        s = r*random()
        points.append((x+s*math.cos(theta), y+s*math.sin(theta)))
    return points

#rand_coord = rand_cluster(n = 500,c = (22.9068, 43.1729), r= (150))


def unique(list1):
     
    # insert the list to the set
    list_set = set(list1)
    # convert the set to the list
    unique_list = (list(list_set))
    name_list = []
    for x in unique_list:
        name_list.append(x)
        
    return name_list
```

location_keys = {
'Quito' : (0.1807, 78.4678),
'Beijing'  : (39.9042, 116.4074),
'Sydney' : (33.8688, 151.2093),
'New York' : (40.7128, 74.006),
'Maimi': (25.7907, 80.1300),
'Rio' : (22.9068, 43.1729)}

use those 5 cities as center to randomly select cities data base



```python
# create a list of target cities coordinates
locationCoords = [(0.1807, 78.4678), (39.9042, 116.4074), (33.8688, 151.2093),(40.7128, 74.006), (25.7907, 80.1300), (22.9068, 43.1729) ]
```


```python

def  get_total_city_name():
    total_list = []
    cities = []
    names = []
    total_coord = []
    for coordIndex in range(0,len(locationCoords)-1) :
        rand_coord = rand_cluster(n = 500,c = locationCoords[coordIndex], r= (500))
        
        for coordinate_pair in rand_coord:
            lat, lon = coordinate_pair
            cities.append(citipy.nearest_city(lat, lon))

        for city in cities:
            name = city.city_name
            names.append(name)
    return total_list
```


```python
# run get_total_city_name() 3 time to get enough random cities
listt= []
for i in range(3):
    listt.append(get_total_city_name())
```


```python
# create a total list of cities and run unique one more time to get unique names of city
finalCityList =listt[0][0] +listt[1][0] + listt[2][0]
finalCityList = unique(finalCityList)
len(finalCityList)
```




    917




```python
# port list into pandas dataframe 
City_Weather = pd.DataFrame(finalCityList)
City_Weather= City_Weather.rename(columns = {0: 'city_name'})
# get 500 randomsample from the dataframe

random_city = City_Weather.sample(n= 700) # set 700 in case some search resulte dont work
random_city.to_csv('random_city.csv', index = False)
```


```python
data = pd.read_csv('random_city.csv')
data.head()
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
      <th>city_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tashla</td>
    </tr>
    <tr>
      <th>1</th>
      <td>vidim</td>
    </tr>
    <tr>
      <th>2</th>
      <td>farafangana</td>
    </tr>
    <tr>
      <th>3</th>
      <td>longyearbyen</td>
    </tr>
    <tr>
      <th>4</th>
      <td>porto novo</td>
    </tr>
  </tbody>
</table>
</div>




```python
base_url = "http://api.openweathermap.org/data/2.5/weather?"

# create new columns 
data['lat']= " "
data['temp']= " "
data['humid']= " "
data['cloud']= " "
data['wind_speed']= " "
row_count = 0
for index,row in data.iterrows() : # lets trial first 
    query = row['city_name']
    query_url = base_url + "appid=" + api_key + "&q=" + query
    
    print("Now retrieving city # " + str(row_count))
    row_count += 1


    responses = requests.get(query_url)
    print(responses.url)
    responses_city = responses.json()
    
    try:
        citi_lat = responses_city['coord']['lat']
        citi_temp = responses_city['main']['temp']
        citi_humid = responses_city['main']['humidity']
        citi_cloud = responses_city['clouds']['all']
        citi_windspeed = responses_city['wind']['speed']

        data.set_value(index, "lat", citi_lat)
        data.set_value(index, "temp", citi_temp)
        data.set_value(index, 'humid', citi_humid)
        data.set_value(index, 'cloud', citi_cloud)
        data.set_value(index, 'wind_speed', citi_windspeed)  
        

    except (KeyError, IndexError):
        
        print("Error with city data. Skipping")
        
        continue


```

    Now retrieving city # 0
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tashla
    Now retrieving city # 1


    /Users/apple/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:29: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    /Users/apple/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:30: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    /Users/apple/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:31: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    /Users/apple/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:32: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    /Users/apple/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:33: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead


    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vidim
    Now retrieving city # 2
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=farafangana
    Now retrieving city # 3
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=longyearbyen
    Now retrieving city # 4
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=porto%20novo
    Now retrieving city # 5
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ismailia
    Now retrieving city # 6
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=seoul
    Now retrieving city # 7
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pojuca
    Now retrieving city # 8
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=siolim
    Error with city data. Skipping
    Now retrieving city # 9
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=carlsbad
    Now retrieving city # 10
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=timra
    Now retrieving city # 11
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jalu
    Now retrieving city # 12
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ancud
    Now retrieving city # 13
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kozluk
    Error with city data. Skipping
    Now retrieving city # 14
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zhigansk
    Now retrieving city # 15
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=batagay-alyta
    Error with city data. Skipping
    Now retrieving city # 16
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mount%20isa
    Now retrieving city # 17
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karamea
    Error with city data. Skipping
    Now retrieving city # 18
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nanortalik
    Now retrieving city # 19
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=saint-philippe
    Now retrieving city # 20
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=angoche
    Now retrieving city # 21
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lukovetskiy
    Now retrieving city # 22
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rizhao
    Now retrieving city # 23
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=petropavl
    Now retrieving city # 24
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aradippou
    Now retrieving city # 25
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kropotkin
    Now retrieving city # 26
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=inhambane
    Now retrieving city # 27
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=laguna
    Now retrieving city # 28
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aksarka
    Now retrieving city # 29
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=arlit
    Now retrieving city # 30
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=huainan
    Now retrieving city # 31
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=along
    Now retrieving city # 32
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bereznik
    Now retrieving city # 33
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=leh
    Now retrieving city # 34
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jiuquan
    Now retrieving city # 35
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=namatanai
    Now retrieving city # 36
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=buala
    Now retrieving city # 37
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kegayli
    Error with city data. Skipping
    Now retrieving city # 38
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=toora-khem
    Now retrieving city # 39
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ramnagar
    Now retrieving city # 40
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=teykovo
    Now retrieving city # 41
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sitka
    Now retrieving city # 42
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jilmah
    Error with city data. Skipping
    Now retrieving city # 43
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=qostanay
    Now retrieving city # 44
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=atuona
    Now retrieving city # 45
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=louisbourg
    Error with city data. Skipping
    Now retrieving city # 46
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bolshiye%20klyuchishchi
    Error with city data. Skipping
    Now retrieving city # 47
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lamu
    Now retrieving city # 48
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=faya
    Now retrieving city # 49
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=verkhnyaya%20inta
    Now retrieving city # 50
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dong%20hoi
    Now retrieving city # 51
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mosquera
    Now retrieving city # 52
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lingyuan
    Now retrieving city # 53
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=diffa
    Now retrieving city # 54
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=luang%20prabang
    Now retrieving city # 55
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=swan%20hill
    Now retrieving city # 56
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vung%20tau
    Now retrieving city # 57
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sinop
    Now retrieving city # 58
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kamenskoye
    Error with city data. Skipping
    Now retrieving city # 59
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=manokwari
    Now retrieving city # 60
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bose
    Now retrieving city # 61
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=batsfjord
    Now retrieving city # 62
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=port%20pirie
    Now retrieving city # 63
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=urumqi
    Error with city data. Skipping
    Now retrieving city # 64
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=changping
    Now retrieving city # 65
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sola
    Now retrieving city # 66
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sao%20francisco%20de%20assis
    Error with city data. Skipping
    Now retrieving city # 67
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mandera
    Now retrieving city # 68
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kalabo
    Now retrieving city # 69
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bacolod
    Now retrieving city # 70
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=poltavka
    Now retrieving city # 71
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=barrow
    Now retrieving city # 72
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=torbay
    Now retrieving city # 73
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=avera
    Now retrieving city # 74
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ucluelet
    Now retrieving city # 75
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=conde
    Now retrieving city # 76
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yatou
    Now retrieving city # 77
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rameswaram
    Now retrieving city # 78
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tari
    Now retrieving city # 79
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kruisfontein
    Now retrieving city # 80
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tezu
    Now retrieving city # 81
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sakakah
    Error with city data. Skipping
    Now retrieving city # 82
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=warangal
    Now retrieving city # 83
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=samarai
    Now retrieving city # 84
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=noumea
    Now retrieving city # 85
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=miandrivazo
    Now retrieving city # 86
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tazovskiy
    Now retrieving city # 87
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bredasdorp
    Now retrieving city # 88
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=changji
    Now retrieving city # 89
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=abu%20kamal
    Now retrieving city # 90
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bitetto
    Now retrieving city # 91
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sorong
    Now retrieving city # 92
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kalianget
    Now retrieving city # 93
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nizwa
    Now retrieving city # 94
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=miyako
    Now retrieving city # 95
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=attawapiskat
    Error with city data. Skipping
    Now retrieving city # 96
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=galle
    Now retrieving city # 97
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=qinhuangdao
    Now retrieving city # 98
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hithadhoo
    Now retrieving city # 99
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=oliver
    Now retrieving city # 100
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=viligili
    Error with city data. Skipping
    Now retrieving city # 101
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=oranjemund
    Now retrieving city # 102
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tilichiki
    Now retrieving city # 103
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hammerfest
    Now retrieving city # 104
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=san%20cristobal
    Now retrieving city # 105
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=xiuyan
    Now retrieving city # 106
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ahipara
    Now retrieving city # 107
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=port%20lincoln
    Now retrieving city # 108
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=totma
    Now retrieving city # 109
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=charlestown
    Now retrieving city # 110
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=palu
    Now retrieving city # 111
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tiksi
    Now retrieving city # 112
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=milkovo
    Now retrieving city # 113
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pevek
    Now retrieving city # 114
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=morant%20bay
    Now retrieving city # 115
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mancio%20lima
    Error with city data. Skipping
    Now retrieving city # 116
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alvorada
    Now retrieving city # 117
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chitral
    Now retrieving city # 118
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=boguchany
    Now retrieving city # 119
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kankan
    Now retrieving city # 120
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kutoarjo
    Now retrieving city # 121
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=daura
    Now retrieving city # 122
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=isangel
    Now retrieving city # 123
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=belaya%20gora
    Now retrieving city # 124
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=taltal
    Now retrieving city # 125
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kigoma
    Now retrieving city # 126
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=monroe
    Now retrieving city # 127
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=inirida
    Now retrieving city # 128
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=morondava
    Now retrieving city # 129
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=birin
    Now retrieving city # 130
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shirgaon
    Error with city data. Skipping
    Now retrieving city # 131
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=oliveirinha
    Now retrieving city # 132
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=najran
    Now retrieving city # 133
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=marsa%20matruh
    Now retrieving city # 134
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mae%20sai
    Now retrieving city # 135
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=la%20poza
    Now retrieving city # 136
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=storforshei
    Now retrieving city # 137
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kavaratti
    Now retrieving city # 138
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=severnyy
    Error with city data. Skipping
    Now retrieving city # 139
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mahuva
    Now retrieving city # 140
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=touros
    Now retrieving city # 141
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dudinka
    Now retrieving city # 142
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ilulissat
    Now retrieving city # 143
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rialma
    Now retrieving city # 144
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dongargaon
    Now retrieving city # 145
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=linxia
    Now retrieving city # 146
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shu
    Now retrieving city # 147
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bilaua
    Error with city data. Skipping
    Now retrieving city # 148
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=energetik
    Now retrieving city # 149
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=broome
    Now retrieving city # 150
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=calumboyan
    Now retrieving city # 151
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hirado
    Now retrieving city # 152
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=balkhash
    Now retrieving city # 153
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=souillac
    Now retrieving city # 154
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chokurdakh
    Now retrieving city # 155
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=douentza
    Now retrieving city # 156
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=turukhansk
    Now retrieving city # 157
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wulanhaote
    Error with city data. Skipping
    Now retrieving city # 158
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=warqla
    Error with city data. Skipping
    Now retrieving city # 159
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pringsewu
    Now retrieving city # 160
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=muros
    Now retrieving city # 161
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=savinka
    Now retrieving city # 162
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zlatoustovsk
    Error with city data. Skipping
    Now retrieving city # 163
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kushima
    Now retrieving city # 164
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=asilah
    Error with city data. Skipping
    Now retrieving city # 165
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=matara
    Now retrieving city # 166
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=merke
    Now retrieving city # 167
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tura
    Now retrieving city # 168
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nhulunbuy
    Now retrieving city # 169
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bengkalis
    Error with city data. Skipping
    Now retrieving city # 170
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=singaraja
    Now retrieving city # 171
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=galiwinku
    Error with city data. Skipping
    Now retrieving city # 172
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bathsheba
    Now retrieving city # 173
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gazojak
    Now retrieving city # 174
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=roald
    Now retrieving city # 175
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ziro
    Now retrieving city # 176
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alizai
    Now retrieving city # 177
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kabrai
    Now retrieving city # 178
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pahalgam
    Now retrieving city # 179
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=amritsar
    Now retrieving city # 180
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shumerlya
    Now retrieving city # 181
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tolaga%20bay
    Now retrieving city # 182
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=helong
    Now retrieving city # 183
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=katha
    Error with city data. Skipping
    Now retrieving city # 184
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pandan
    Now retrieving city # 185
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=groningen
    Now retrieving city # 186
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bembereke
    Now retrieving city # 187
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bindura
    Now retrieving city # 188
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ronne
    Now retrieving city # 189
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vanimo
    Now retrieving city # 190
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mujiayingzi
    Now retrieving city # 191
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=oktyabrskoye
    Now retrieving city # 192
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=batagay
    Now retrieving city # 193
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=faanui
    Now retrieving city # 194
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=moron
    Now retrieving city # 195
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kawambwa
    Now retrieving city # 196
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=teshie
    Now retrieving city # 197
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ngukurr
    Error with city data. Skipping
    Now retrieving city # 198
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=new%20norfolk
    Now retrieving city # 199
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dire%20dawa
    Now retrieving city # 200
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lasa
    Now retrieving city # 201
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=banda%20aceh
    Now retrieving city # 202
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mackay
    Now retrieving city # 203
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lanuza
    Now retrieving city # 204
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kalmunai
    Now retrieving city # 205
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=palauig
    Now retrieving city # 206
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ginda
    Now retrieving city # 207
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=natitingou
    Now retrieving city # 208
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=petropavlovsk-kamchatskiy
    Now retrieving city # 209
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bengkulu
    Error with city data. Skipping
    Now retrieving city # 210
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=resistencia
    Now retrieving city # 211
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=xining
    Now retrieving city # 212
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nemuro
    Now retrieving city # 213
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=cockburn%20town
    Now retrieving city # 214
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kavieng
    Now retrieving city # 215
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=anito
    Now retrieving city # 216
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bang%20saphan
    Now retrieving city # 217
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=asau
    Error with city data. Skipping
    Now retrieving city # 218
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sheregesh
    Now retrieving city # 219
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=otane
    Now retrieving city # 220
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vikhorevka
    Now retrieving city # 221
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=de-kastri
    Now retrieving city # 222
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=maracana
    Error with city data. Skipping
    Now retrieving city # 223
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aklavik
    Now retrieving city # 224
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=okhotsk
    Now retrieving city # 225
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=fevralsk
    Error with city data. Skipping
    Now retrieving city # 226
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kendari
    Now retrieving city # 227
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=suhbaatar
    Now retrieving city # 228
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kawalu
    Now retrieving city # 229
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=luderitz
    Now retrieving city # 230
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chulman
    Now retrieving city # 231
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hambantota
    Now retrieving city # 232
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=boende
    Now retrieving city # 233
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kosovo%20polje
    Now retrieving city # 234
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sistranda
    Now retrieving city # 235
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=berlevag
    Now retrieving city # 236
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=taksimo
    Now retrieving city # 237
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=znamenskoye
    Now retrieving city # 238
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karacakoy
    Now retrieving city # 239
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=stalowa%20wola
    Now retrieving city # 240
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=belyy%20yar
    Now retrieving city # 241
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hermanus
    Now retrieving city # 242
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=verkhnevilyuysk
    Now retrieving city # 243
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=prokudskoye
    Now retrieving city # 244
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zenzeli
    Now retrieving city # 245
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=capao%20da%20canoa
    Now retrieving city # 246
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=srednekolymsk
    Now retrieving city # 247
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sinalunga
    Now retrieving city # 248
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=goderich
    Now retrieving city # 249
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kadykchan
    Error with city data. Skipping
    Now retrieving city # 250
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mount%20gambier
    Now retrieving city # 251
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=havoysund
    Now retrieving city # 252
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=postojna
    Now retrieving city # 253
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=krasnoselkup
    Error with city data. Skipping
    Now retrieving city # 254
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=parabel
    Now retrieving city # 255
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=port%20hedland
    Now retrieving city # 256
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=odweyne
    Error with city data. Skipping
    Now retrieving city # 257
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=georgetown
    Now retrieving city # 258
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=amta
    Now retrieving city # 259
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ca%20mau
    Now retrieving city # 260
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=maruko
    Now retrieving city # 261
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=coripata
    Now retrieving city # 262
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kirakira
    Now retrieving city # 263
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=eregli
    Now retrieving city # 264
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=albany
    Now retrieving city # 265
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pozo%20colorado
    Now retrieving city # 266
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sirjan
    Now retrieving city # 267
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hirara
    Now retrieving city # 268
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kyshtovka
    Now retrieving city # 269
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=guarda
    Now retrieving city # 270
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bandarbeyla
    Now retrieving city # 271
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=galesong
    Now retrieving city # 272
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dzilam%20gonzalez
    Now retrieving city # 273
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=raga
    Error with city data. Skipping
    Now retrieving city # 274
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shache
    Now retrieving city # 275
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=anuradhapura
    Now retrieving city # 276
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=peking
    Now retrieving city # 277
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=taga
    Now retrieving city # 278
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vanavara
    Now retrieving city # 279
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hamilton
    Now retrieving city # 280
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=blagoyevo
    Now retrieving city # 281
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karamay
    Error with city data. Skipping
    Now retrieving city # 282
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kungurtug
    Now retrieving city # 283
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mangan
    Now retrieving city # 284
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dongsheng
    Now retrieving city # 285
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lata
    Now retrieving city # 286
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ningbo
    Now retrieving city # 287
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zhangjiakou
    Now retrieving city # 288
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bambous%20virieux
    Now retrieving city # 289
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=los%20llanos%20de%20aridane
    Now retrieving city # 290
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=adilabad
    Now retrieving city # 291
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mochishche
    Now retrieving city # 292
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=illoqqortoormiut
    Error with city data. Skipping
    Now retrieving city # 293
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dhidhdhoo
    Now retrieving city # 294
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ribeira%20grande
    Now retrieving city # 295
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shirokiy
    Now retrieving city # 296
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=panji
    Now retrieving city # 297
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sidhi
    Now retrieving city # 298
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karauli
    Now retrieving city # 299
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mwinilunga
    Now retrieving city # 300
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=floro
    Now retrieving city # 301
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kandrian
    Now retrieving city # 302
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shestakovo
    Now retrieving city # 303
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tongzhou
    Now retrieving city # 304
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=veraval
    Now retrieving city # 305
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dedougou
    Now retrieving city # 306
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kirkenes
    Now retrieving city # 307
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pyaozerskiy
    Now retrieving city # 308
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lazarev
    Now retrieving city # 309
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=broken%20hill
    Now retrieving city # 310
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yanan
    Error with city data. Skipping
    Now retrieving city # 311
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zyryanka
    Now retrieving city # 312
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karakendzha
    Error with city data. Skipping
    Now retrieving city # 313
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rikitea
    Now retrieving city # 314
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pundaguitan
    Now retrieving city # 315
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=eppingen
    Now retrieving city # 316
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nome
    Now retrieving city # 317
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sambava
    Now retrieving city # 318
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=purbalingga
    Now retrieving city # 319
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=port%20blair
    Now retrieving city # 320
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kadhan
    Now retrieving city # 321
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=haibowan
    Error with city data. Skipping
    Now retrieving city # 322
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=russell
    Now retrieving city # 323
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=forbes
    Now retrieving city # 324
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mildura
    Now retrieving city # 325
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=margate
    Now retrieving city # 326
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bay-khaak
    Now retrieving city # 327
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=poya
    Now retrieving city # 328
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kurumkan
    Now retrieving city # 329
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nizhnevartovsk
    Now retrieving city # 330
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=catuday
    Now retrieving city # 331
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=abu%20dhabi
    Now retrieving city # 332
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sibu
    Now retrieving city # 333
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tomaszow%20lubelski
    Now retrieving city # 334
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kodiak
    Now retrieving city # 335
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=port%20elizabeth
    Now retrieving city # 336
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sicuani
    Now retrieving city # 337
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=muzaffarabad
    Now retrieving city # 338
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=krapivinskiy
    Now retrieving city # 339
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=san%20policarpo
    Now retrieving city # 340
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ejura
    Now retrieving city # 341
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=naze
    Now retrieving city # 342
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=marang
    Error with city data. Skipping
    Now retrieving city # 343
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=moindou
    Now retrieving city # 344
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ritchie
    Now retrieving city # 345
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nenagh
    Now retrieving city # 346
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=esperance
    Now retrieving city # 347
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=eyl
    Now retrieving city # 348
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=klaksvik
    Now retrieving city # 349
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=camabatela
    Now retrieving city # 350
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jiayuguan
    Now retrieving city # 351
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hofn
    Now retrieving city # 352
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=malayal
    Now retrieving city # 353
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=srandakan
    Now retrieving city # 354
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yumen
    Now retrieving city # 355
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lashio
    Now retrieving city # 356
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=thinadhoo
    Now retrieving city # 357
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kargil
    Now retrieving city # 358
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nyurba
    Now retrieving city # 359
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pangnirtung
    Now retrieving city # 360
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=narasannapeta
    Now retrieving city # 361
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alipur
    Now retrieving city # 362
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=iganga
    Now retrieving city # 363
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tumannyy
    Error with city data. Skipping
    Now retrieving city # 364
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zurrieq
    Now retrieving city # 365
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mocambique
    Error with city data. Skipping
    Now retrieving city # 366
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sao%20joao%20da%20barra
    Now retrieving city # 367
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=manturovo
    Now retrieving city # 368
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wukari
    Now retrieving city # 369
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ishigaki
    Now retrieving city # 370
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tarija
    Now retrieving city # 371
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=simao
    Now retrieving city # 372
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bolsheustikinskoye
    Error with city data. Skipping
    Now retrieving city # 373
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kuche
    Error with city data. Skipping
    Now retrieving city # 374
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=duiwelskloof
    Now retrieving city # 375
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pulandian
    Now retrieving city # 376
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bokspits
    Error with city data. Skipping
    Now retrieving city # 377
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=cidreira
    Now retrieving city # 378
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sanand
    Now retrieving city # 379
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=choya
    Now retrieving city # 380
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tortoli
    Now retrieving city # 381
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ust-nera
    Now retrieving city # 382
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=den%20helder
    Now retrieving city # 383
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jabiru
    Error with city data. Skipping
    Now retrieving city # 384
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dhule
    Now retrieving city # 385
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ippy
    Now retrieving city # 386
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=raudeberg
    Now retrieving city # 387
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bethel
    Now retrieving city # 388
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=holme
    Now retrieving city # 389
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=payo
    Now retrieving city # 390
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mahoba
    Now retrieving city # 391
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ambon
    Now retrieving city # 392
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=toliary
    Error with city data. Skipping
    Now retrieving city # 393
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=upernavik
    Now retrieving city # 394
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=namibe
    Now retrieving city # 395
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=havre
    Now retrieving city # 396
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=katsuura
    Now retrieving city # 397
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chiredzi
    Now retrieving city # 398
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gizo
    Now retrieving city # 399
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=darhan
    Now retrieving city # 400
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=fortuna
    Now retrieving city # 401
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sentyabrskiy
    Error with city data. Skipping
    Now retrieving city # 402
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sangar
    Now retrieving city # 403
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=andarab
    Error with city data. Skipping
    Now retrieving city # 404
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shiraz
    Now retrieving city # 405
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pousat
    Error with city data. Skipping
    Now retrieving city # 406
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nazarovo
    Now retrieving city # 407
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=avare
    Now retrieving city # 408
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yellowknife
    Now retrieving city # 409
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mogzon
    Now retrieving city # 410
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=fengxian
    Now retrieving city # 411
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=cabo%20san%20lucas
    Now retrieving city # 412
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=qandahar
    Error with city data. Skipping
    Now retrieving city # 413
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tasbuget
    Error with city data. Skipping
    Now retrieving city # 414
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zaysan
    Now retrieving city # 415
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sorata
    Now retrieving city # 416
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=busselton
    Now retrieving city # 417
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=svetlyy
    Error with city data. Skipping
    Now retrieving city # 418
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=half%20moon%20bay
    Now retrieving city # 419
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=thoen
    Now retrieving city # 420
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=birjand
    Now retrieving city # 421
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kashi
    Error with city data. Skipping
    Now retrieving city # 422
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=salalah
    Now retrieving city # 423
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=juarez
    Now retrieving city # 424
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=norman%20wells
    Now retrieving city # 425
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=velikiy%20ustyug
    Now retrieving city # 426
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bunia
    Now retrieving city # 427
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mahendranagar
    Now retrieving city # 428
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=asfi
    Error with city data. Skipping
    Now retrieving city # 429
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ulety
    Now retrieving city # 430
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=saint%20george
    Now retrieving city # 431
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=geraldton
    Now retrieving city # 432
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=troitsko-pechorsk
    Now retrieving city # 433
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kulhudhuffushi
    Now retrieving city # 434
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tawkar
    Error with city data. Skipping
    Now retrieving city # 435
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mahibadhoo
    Now retrieving city # 436
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=berezovyy
    Now retrieving city # 437
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=antalaha
    Now retrieving city # 438
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shimoda
    Now retrieving city # 439
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=poronaysk
    Now retrieving city # 440
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=christchurch
    Now retrieving city # 441
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=khanpur
    Now retrieving city # 442
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yakeshi
    Now retrieving city # 443
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=adrian
    Now retrieving city # 444
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gejiu
    Now retrieving city # 445
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pasighat
    Now retrieving city # 446
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sinnamary
    Now retrieving city # 447
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=clyde%20river
    Now retrieving city # 448
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=le%20port
    Now retrieving city # 449
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=amga
    Now retrieving city # 450
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sohag
    Now retrieving city # 451
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aysha
    Now retrieving city # 452
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ajdabiya
    Now retrieving city # 453
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mon
    Now retrieving city # 454
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=whitehorse
    Now retrieving city # 455
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=taolanaro
    Error with city data. Skipping
    Now retrieving city # 456
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nurota
    Now retrieving city # 457
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=acapulco
    Now retrieving city # 458
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=abonnema
    Now retrieving city # 459
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=baiyin
    Now retrieving city # 460
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gandai
    Now retrieving city # 461
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=halalo
    Error with city data. Skipping
    Now retrieving city # 462
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=teya
    Now retrieving city # 463
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tahoua
    Now retrieving city # 464
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ginir
    Now retrieving city # 465
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=moundou
    Now retrieving city # 466
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yar-sale
    Now retrieving city # 467
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shemonaikha
    Now retrieving city # 468
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tautira
    Now retrieving city # 469
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=auki
    Now retrieving city # 470
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gashua
    Now retrieving city # 471
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=davila
    Now retrieving city # 472
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=beringovskiy
    Now retrieving city # 473
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=takoradi
    Now retrieving city # 474
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ruatoria
    Error with city data. Skipping
    Now retrieving city # 475
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hami
    Now retrieving city # 476
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wattegama
    Now retrieving city # 477
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=abatskoye
    Now retrieving city # 478
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=paradwip
    Error with city data. Skipping
    Now retrieving city # 479
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=andra
    Now retrieving city # 480
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=niamey
    Now retrieving city # 481
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dingle
    Now retrieving city # 482
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=channel-port%20aux%20basques
    Now retrieving city # 483
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mehamn
    Now retrieving city # 484
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=butaritari
    Now retrieving city # 485
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=phangnga
    Now retrieving city # 486
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nianzishan
    Now retrieving city # 487
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nikolskoye
    Now retrieving city # 488
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gogapur
    Now retrieving city # 489
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karkaralinsk
    Error with city data. Skipping
    Now retrieving city # 490
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=menongue
    Now retrieving city # 491
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sabzevar
    Now retrieving city # 492
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aswan
    Now retrieving city # 493
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=brae
    Now retrieving city # 494
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=winsum
    Now retrieving city # 495
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=itarema
    Now retrieving city # 496
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wajima
    Now retrieving city # 497
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dwarka
    Now retrieving city # 498
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lorengau
    Now retrieving city # 499
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=muhos
    Now retrieving city # 500
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ponta%20do%20sol
    Now retrieving city # 501
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=airai
    Now retrieving city # 502
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sept-iles
    Now retrieving city # 503
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=veshenskaya
    Now retrieving city # 504
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bahua
    Now retrieving city # 505
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kattivakkam
    Now retrieving city # 506
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=komsomolskiy
    Now retrieving city # 507
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=stepnyak
    Now retrieving city # 508
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=avarua
    Now retrieving city # 509
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chotila
    Now retrieving city # 510
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=karauzyak
    Error with city data. Skipping
    Now retrieving city # 511
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=northam
    Now retrieving city # 512
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sao%20filipe
    Now retrieving city # 513
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=barbar
    Error with city data. Skipping
    Now retrieving city # 514
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=haimen
    Now retrieving city # 515
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mandiana
    Now retrieving city # 516
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wajid
    Now retrieving city # 517
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nerchinskiy%20zavod
    Now retrieving city # 518
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rungata
    Error with city data. Skipping
    Now retrieving city # 519
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tarudant
    Error with city data. Skipping
    Now retrieving city # 520
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dawson%20creek
    Now retrieving city # 521
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=terrace
    Now retrieving city # 522
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=haines%20junction
    Now retrieving city # 523
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kudahuvadhoo
    Now retrieving city # 524
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=port-gentil
    Now retrieving city # 525
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=majene
    Now retrieving city # 526
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=moroni
    Now retrieving city # 527
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=qingquan
    Now retrieving city # 528
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=marawi
    Now retrieving city # 529
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=buchanan
    Now retrieving city # 530
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lander
    Now retrieving city # 531
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=coquimbo
    Now retrieving city # 532
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=xuanhua
    Now retrieving city # 533
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yuanping
    Now retrieving city # 534
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sabalgarh
    Now retrieving city # 535
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hohhot
    Now retrieving city # 536
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=adrar
    Now retrieving city # 537
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=boyolangu
    Now retrieving city # 538
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gangtok
    Now retrieving city # 539
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=husavik
    Now retrieving city # 540
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=buqayq
    Error with city data. Skipping
    Now retrieving city # 541
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=saskylakh
    Now retrieving city # 542
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=punta%20arenas
    Now retrieving city # 543
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aleppo
    Now retrieving city # 544
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=itoman
    Now retrieving city # 545
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kutum
    Now retrieving city # 546
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gap
    Now retrieving city # 547
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=phalodi
    Now retrieving city # 548
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jiddah
    Error with city data. Skipping
    Now retrieving city # 549
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=manihari
    Now retrieving city # 550
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=severo-kurilsk
    Now retrieving city # 551
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=laizhou
    Now retrieving city # 552
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kiama
    Now retrieving city # 553
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=soyo
    Now retrieving city # 554
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=atasu
    Now retrieving city # 555
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=palabuhanratu
    Error with city data. Skipping
    Now retrieving city # 556
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ostrovnoy
    Now retrieving city # 557
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=deputatskiy
    Now retrieving city # 558
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=verkhovye
    Now retrieving city # 559
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=darab
    Now retrieving city # 560
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=morehead
    Now retrieving city # 561
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sheopur
    Now retrieving city # 562
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bluff
    Now retrieving city # 563
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=falam
    Now retrieving city # 564
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=petukhovo
    Now retrieving city # 565
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=turtkul
    Error with city data. Skipping
    Now retrieving city # 566
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kishtwar
    Now retrieving city # 567
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=plastun
    Now retrieving city # 568
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=samur
    Now retrieving city # 569
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dimbokro
    Now retrieving city # 570
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=egvekinot
    Now retrieving city # 571
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tuktoyaktuk
    Now retrieving city # 572
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wajir
    Now retrieving city # 573
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chuy
    Now retrieving city # 574
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=artyom
    Now retrieving city # 575
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=malwan
    Error with city data. Skipping
    Now retrieving city # 576
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chandbali
    Now retrieving city # 577
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=phan%20rang
    Error with city data. Skipping
    Now retrieving city # 578
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=okha
    Now retrieving city # 579
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=babanusah
    Error with city data. Skipping
    Now retrieving city # 580
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=luena
    Now retrieving city # 581
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=provideniya
    Now retrieving city # 582
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hanzhong
    Now retrieving city # 583
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=albanel
    Now retrieving city # 584
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=anjozorobe
    Now retrieving city # 585
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=puerto%20escondido
    Now retrieving city # 586
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=east%20london
    Now retrieving city # 587
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=labutta
    Error with city data. Skipping
    Now retrieving city # 588
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=surin
    Now retrieving city # 589
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=buraydah
    Now retrieving city # 590
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=wanning
    Now retrieving city # 591
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=orumiyeh
    Now retrieving city # 592
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=harper
    Now retrieving city # 593
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=maymyo
    Now retrieving city # 594
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=quzhou
    Now retrieving city # 595
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lang%20suan
    Now retrieving city # 596
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=prado
    Now retrieving city # 597
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vaini
    Now retrieving city # 598
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bolobo
    Now retrieving city # 599
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=shwebo
    Now retrieving city # 600
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kaseda
    Now retrieving city # 601
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mangrol
    Now retrieving city # 602
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sabang
    Now retrieving city # 603
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alofi
    Now retrieving city # 604
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=aksu
    Now retrieving city # 605
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ust-barguzin
    Now retrieving city # 606
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ploemeur
    Now retrieving city # 607
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dindori
    Now retrieving city # 608
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zhangye
    Now retrieving city # 609
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jahrom
    Error with city data. Skipping
    Now retrieving city # 610
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tual
    Now retrieving city # 611
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kloulklubed
    Now retrieving city # 612
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=nouadhibou
    Now retrieving city # 613
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=akhmim
    Now retrieving city # 614
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=taoudenni
    Now retrieving city # 615
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alibag
    Now retrieving city # 616
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mehran
    Now retrieving city # 617
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=umzimvubu
    Error with city data. Skipping
    Now retrieving city # 618
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zhanatas
    Error with city data. Skipping
    Now retrieving city # 619
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sarahan
    Now retrieving city # 620
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kamaishi
    Now retrieving city # 621
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yeniseysk
    Now retrieving city # 622
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=saint-francois
    Now retrieving city # 623
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pingliang
    Now retrieving city # 624
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=jhinjhak
    Now retrieving city # 625
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=arman
    Now retrieving city # 626
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=muriwai%20beach
    Now retrieving city # 627
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yulara
    Now retrieving city # 628
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tongliao
    Now retrieving city # 629
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=constitucion
    Now retrieving city # 630
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=baruun-urt
    Now retrieving city # 631
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=honiara
    Now retrieving city # 632
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tsihombe
    Error with city data. Skipping
    Now retrieving city # 633
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=narauis
    Error with city data. Skipping
    Now retrieving city # 634
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=meiganga
    Now retrieving city # 635
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pasarkemis
    Now retrieving city # 636
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=harij
    Now retrieving city # 637
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=brahmapuri
    Error with city data. Skipping
    Now retrieving city # 638
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yamada
    Now retrieving city # 639
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=omboue
    Now retrieving city # 640
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sibolga
    Now retrieving city # 641
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kysyl-syr
    Now retrieving city # 642
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=pietarsaari
    Error with city data. Skipping
    Now retrieving city # 643
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bundaberg
    Now retrieving city # 644
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tasiilaq
    Now retrieving city # 645
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=puro
    Now retrieving city # 646
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tanout
    Now retrieving city # 647
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=valentin%20gomez%20farias
    Now retrieving city # 648
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sribne
    Now retrieving city # 649
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yanam
    Now retrieving city # 650
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dicabisagan
    Now retrieving city # 651
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lolua
    Error with city data. Skipping
    Now retrieving city # 652
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kencong
    Now retrieving city # 653
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lodhran
    Now retrieving city # 654
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=zhezkazgan
    Now retrieving city # 655
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dali
    Now retrieving city # 656
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=cherskiy
    Now retrieving city # 657
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=cape%20town
    Now retrieving city # 658
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bajil
    Now retrieving city # 659
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yilan
    Now retrieving city # 660
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=labytnangi
    Now retrieving city # 661
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=bandar-e%20lengeh
    Now retrieving city # 662
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=podor
    Now retrieving city # 663
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=victoria
    Now retrieving city # 664
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kirovskiy
    Now retrieving city # 665
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=marzuq
    Now retrieving city # 666
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tottori
    Now retrieving city # 667
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rumoi
    Now retrieving city # 668
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=vostok
    Now retrieving city # 669
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yoichi
    Now retrieving city # 670
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=anadyr
    Now retrieving city # 671
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=madison
    Now retrieving city # 672
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=hudiksvall
    Now retrieving city # 673
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dikson
    Now retrieving city # 674
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=khor
    Now retrieving city # 675
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gediz
    Now retrieving city # 676
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=yerbogachen
    Now retrieving city # 677
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=palana
    Now retrieving city # 678
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=huilong
    Now retrieving city # 679
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=kagadi
    Now retrieving city # 680
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sjovegan
    Now retrieving city # 681
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=cabedelo
    Now retrieving city # 682
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=lyangasovo
    Now retrieving city # 683
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ushtobe
    Now retrieving city # 684
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=sorland
    Now retrieving city # 685
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=leningradskiy
    Now retrieving city # 686
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=rawson
    Now retrieving city # 687
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=padampur
    Now retrieving city # 688
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=minab
    Now retrieving city # 689
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=tunghsiao
    Error with city data. Skipping
    Now retrieving city # 690
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=chumikan
    Now retrieving city # 691
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=romanovka
    Now retrieving city # 692
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alyangula
    Now retrieving city # 693
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=gamba
    Now retrieving city # 694
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=umea
    Now retrieving city # 695
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=alice%20springs
    Now retrieving city # 696
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=umm%20kaddadah
    Now retrieving city # 697
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=dandong
    Now retrieving city # 698
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=mikhaylovka
    Now retrieving city # 699
    http://api.openweathermap.org/data/2.5/weather?appid=dcd88eeb3c71466f340dd27aabe01aae&q=ushibuka



```python
#save data into csv and clean the data with blank value
data = data.replace('NA', ' ').reset_index()
#data.to_csv('city_info.csv', index= False)
```


```python
# import data for visualization
df = pd.read_csv('city_info.csv')
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
      <th>city_name</th>
      <th>lat</th>
      <th>temp</th>
      <th>humid</th>
      <th>cloud</th>
      <th>wind_speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tashla</td>
      <td>53.71</td>
      <td>259.150</td>
      <td>65</td>
      <td>0</td>
      <td>5.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>vidim</td>
      <td>50.47</td>
      <td>273.150</td>
      <td>99</td>
      <td>90</td>
      <td>2.60</td>
    </tr>
    <tr>
      <th>2</th>
      <td>farafangana</td>
      <td>-22.82</td>
      <td>298.479</td>
      <td>100</td>
      <td>64</td>
      <td>8.81</td>
    </tr>
    <tr>
      <th>3</th>
      <td>longyearbyen</td>
      <td>78.22</td>
      <td>260.150</td>
      <td>55</td>
      <td>20</td>
      <td>5.10</td>
    </tr>
    <tr>
      <th>4</th>
      <td>porto novo</td>
      <td>-23.68</td>
      <td>299.254</td>
      <td>63</td>
      <td>0</td>
      <td>2.76</td>
    </tr>
  </tbody>
</table>
</div>




```python
min(df['temp'])
#max(df[''])
```




    237.15400000000002




```python
import matplotlib.pyplot as plt


# Build a scatter plot for each data type
plt.figure(figsize = (10,10))
plt.scatter(x = df["lat"], y = df["temp"], edgecolor="g", linewidths=1, marker="o",
            alpha=0.3, )

# Incorporate the other graph properties
plt.title("Latitude vs Temperature ", fontsize = 23)
plt.ylabel("Temperature", fontsize = 19)
plt.xlabel("Latitude", fontsize = 19)

plt.xlim([min(df['lat']) -20, max(df['lat']) +20])
plt.ylim([min(df['temp']) -10, max(df['temp']) +5])

# Save the figure
#plt.savefig("output_analysis/Population_BankCount.png")

# Show plot
plt.show()

           

```


![png](output_14_0.png)



```python
plt.figure(figsize = (10,10))
plt.scatter(x = df["lat"], y = df["humid"], edgecolor="black", linewidths=1, marker="o",
            alpha=0.3, )

# Incorporate the other graph properties
plt.title("Latitude vs Humidity ", fontsize = 23)
plt.ylabel("Humidity", fontsize = 19)
plt.xlabel("Latitude", fontsize = 19)

plt.xlim([min(df['lat']) -20, max(df['lat']) +20])
plt.ylim([min(df['humid']) -10, max(df['humid']) +4])

# Save the figure
#plt.savefig("output_analysis/Population_BankCount.png")

# Show plot
plt.show()

           
```


![png](output_15_0.png)



```python
plt.figure(figsize = (10,10))
plt.scatter(x = df["lat"], y = df["cloud"], edgecolor="purple", linewidths=1, marker="o",
            alpha=0.3, )

# Incorporate the other graph properties
plt.title("Latitude vs Cloudiness ", fontsize = 23)
plt.ylabel("Clodiness", fontsize = 19)
plt.xlabel("Latitude", fontsize = 19)

plt.xlim([min(df['lat']) -20, max(df['lat']) +20])
plt.ylim([min(df["cloud"]) -10, max(df["cloud"]) +20])

# Save the figure
#plt.savefig("output_analysis/Population_BankCount.png")

# Show plot
plt.show()


```


![png](output_16_0.png)



```python
# Build a scatter plot for each data type
plt.figure(figsize = (10,10))
plt.scatter(x = df["lat"], y = df["wind_speed"], edgecolor="red", linewidths=1, marker="o",
            alpha=0.3, )

# Incorporate the other graph properties
plt.title("Latitude vs Wind Speed ", fontsize = 23)
plt.ylabel("Wind Speed", fontsize = 19)
plt.xlabel("Latitude", fontsize = 19)

plt.xlim([min(df['lat']) -20, max(df['lat']) +20])
plt.ylim([min(df["wind_speed"]) -2, max(df["wind_speed"]) +2])

# Save the figure
#plt.savefig("output_analysis/Population_BankCount.png")

# Show plot
plt.show()
```


![png](output_17_0.png)


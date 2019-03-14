
# WeatherPy
## Analysis
* The closer you get to the equator the warmer it gets regardless of Earth's tilt. Earth's tilt will change the weather for either [> 0 degrees latitude >] positions on Earth, effectively creating seasons.
* The equator seems to have less wind speed than other locations on Earth but the difference is minimal.
* Cloudiness and humidity ranges in cities aren't affected by their proximity to the earth's equator. I would assume that those values are affected by their closeness to bodies of water, this is only a hypothesis and needs to be explored further.


```python
# Dependencies
import csv
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import openweathermapy.core as ow
from datetime import datetime
from citipy import citipy
from progressbar import ProgressBar
```


```python
# latitude range is (-90,90) longitude range is (-180,180)
lat_coord = np.random.uniform(-90,91,700)
lng_coord = np.random.uniform(-180,181,700)
```

## Generate Cities Dataframe


```python
weather_df = pd.DataFrame()
weather_df['city_name'] = ''
weather_df['country_code'] = ''
weather_df['latitude'] = ''
weather_df['longitude'] = ''

bar = ProgressBar()
for i in bar(range(len(lat_coord))):
    weather_df.set_value(i,'city_name',citipy.nearest_city(lat_coord[i], lng_coord[i]).city_name)
    weather_df.set_value(i,'country_code',citipy.nearest_city(lat_coord[i], lng_coord[i]).country_code)
    weather_df.set_value(i,'latitude',round(lat_coord[i],4))
    weather_df.set_value(i,'longitude',round(lng_coord[i],4))
```

    100% (700 of 700) |#######################| Elapsed Time: 0:00:02 Time: 0:00:02



```python
weather_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city_name</th>
      <th>country_code</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>husavik</td>
      <td>is</td>
      <td>80.4436</td>
      <td>-6.3406</td>
    </tr>
    <tr>
      <th>1</th>
      <td>panama city</td>
      <td>us</td>
      <td>28.2604</td>
      <td>-86.2812</td>
    </tr>
    <tr>
      <th>2</th>
      <td>saint-augustin</td>
      <td>ca</td>
      <td>53.8406</td>
      <td>-59.6549</td>
    </tr>
    <tr>
      <th>3</th>
      <td>cayenne</td>
      <td>gf</td>
      <td>12.162</td>
      <td>-46.8953</td>
    </tr>
    <tr>
      <th>4</th>
      <td>cherskiy</td>
      <td>ru</td>
      <td>89.2911</td>
      <td>161.401</td>
    </tr>
  </tbody>
</table>
</div>



## Perform Weather API Calls


```python
# Create settings dictionary with information we're interested in
api_key = "7000bf4dfa565802d1a0bc7afae50bec"
settings = {"units": "imperial", "appid": api_key}
```

All url references to the recorded city-weather data is in the **data_path.csv** file.


```python
weather_df['date'] = ''
weather_df['max_temp'] = ''
weather_df['humidity'] = ''
weather_df['cloudiness'] = ''
weather_df['wind_speed'] = ''

to_csv1 = []
to_csv2 = []

count = 1
url = 'http://api.openweathermap.org/data/2.5/weather?'

progress = ProgressBar(max_value=len(lat_coord)).start()
for index, row in weather_df.iterrows():
    try:
        weather_data = ow.get_current("{},{}".format(row['city_name'],
            row['country_code']),**settings)

        weather_df.set_value(index, 'date', weather_data('dt'))
        weather_df.set_value(index, 'max_temp', weather_data('main.temp_max'))
        weather_df.set_value(index, 'humidity', weather_data('main.humidity'))
        weather_df.set_value(index, 'cloudiness', weather_data('clouds.all'))
        weather_df.set_value(index, 'wind_speed', weather_data('wind.speed'))

        to_csv1.append("Record {} of {} | {}"\
            .format(count, len(weather_df), row['city_name']))
        to_csv2.append("{}APPID={}&units={}&q={},{}"\
            .format(url, api_key, settings['units'], row['city_name'], row['country_code']))

        count += 1
        progress.update(index+1)

    except:
        weather_df.set_value(index, 'date', np.nan)
        weather_df.set_value(index, 'max_temp', np.nan)
        weather_df.set_value(index, 'humidity', np.nan)
        weather_df.set_value(index, 'cloudiness', np.nan)
        weather_df.set_value(index, 'wind_speed', np.nan)

        count += 1
        progress.update(index+1)

progress.finish()

# write data url's to csv
csv_out = open('data_urlPaths.csv','w')
writer = csv.writer(csv_out)
for row in zip(to_csv1,to_csv2):
    writer.writerow(row)
csv_out.close()
```

    100% (700 of 700) |#######################| Elapsed Time: 0:04:11 Time: 0:04:11



```python
# remove nan rows
weather_df = weather_df.dropna()
weather_df.count()
```




    city_name       609
    country_code    609
    latitude        609
    longitude       609
    date            609
    max_temp        609
    humidity        609
    cloudiness      609
    wind_speed      609
    dtype: int64




```python
# display ecuator weather dataframe
weather_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city_name</th>
      <th>country_code</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>date</th>
      <th>max_temp</th>
      <th>humidity</th>
      <th>cloudiness</th>
      <th>wind_speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>husavik</td>
      <td>is</td>
      <td>80.4436</td>
      <td>-6.3406</td>
      <td>1513627200</td>
      <td>44.6</td>
      <td>81</td>
      <td>80</td>
      <td>17.22</td>
    </tr>
    <tr>
      <th>1</th>
      <td>panama city</td>
      <td>us</td>
      <td>28.2604</td>
      <td>-86.2812</td>
      <td>1513626960</td>
      <td>77</td>
      <td>88</td>
      <td>90</td>
      <td>9.17</td>
    </tr>
    <tr>
      <th>2</th>
      <td>saint-augustin</td>
      <td>ca</td>
      <td>53.8406</td>
      <td>-59.6549</td>
      <td>1513629101</td>
      <td>7.99</td>
      <td>100</td>
      <td>12</td>
      <td>13.56</td>
    </tr>
    <tr>
      <th>3</th>
      <td>cayenne</td>
      <td>gf</td>
      <td>12.162</td>
      <td>-46.8953</td>
      <td>1513627200</td>
      <td>82.4</td>
      <td>74</td>
      <td>40</td>
      <td>9.17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>cherskiy</td>
      <td>ru</td>
      <td>89.2911</td>
      <td>161.401</td>
      <td>1513629102</td>
      <td>-18.7</td>
      <td>80</td>
      <td>32</td>
      <td>1.77</td>
    </tr>
  </tbody>
</table>
</div>



## Latitude vs Temperature Plot


```python
plot_date = datetime.fromtimestamp(weather_df['date'][0])\
                    .strftime('%Y-%m-%d %H:%M:%S').split()[0]
```


```python
fig,ax = plt.subplots(figsize=(14,12))

scatter = ax.scatter(weather_df['latitude'],weather_df['max_temp'],
           c=weather_df['max_temp'], cmap=plt.cm.coolwarm, zorder=2)

fig.colorbar(scatter)

ax.set_title('City Latitude vs Max Temperature ({})'.format(plot_date), fontdict={"fontsize":18})
ax.set_xlabel('Latitude', fontdict={"fontsize":16})
ax.set_ylabel('Max Temperature (F)', fontdict={"fontsize":16})
ax.set_xlim(-100,100)
ax.set_ylim(-100,150)
ax.vlines(0,-100,150)
ax.text(-5,0,'equator', rotation=90, fontdict={"fontsize":14})
ax.grid()

plt.show()
```


![png](../images/output_14_0.png)


## Latitude vs Humidity Plot


```python
fig,ax = plt.subplots(figsize=(10,6))
ax.scatter(weather_df['latitude'],weather_df['humidity'],
           c=weather_df['humidity'], cmap=plt.cm.Blues, zorder=2)
ax.set_title('City Latitude vs Humidity ({})'.format(plot_date), fontdict={"fontsize":16})
ax.set_xlabel('Latitude', fontdict={"fontsize":14})
ax.set_ylabel('Humidity (%)', fontdict={"fontsize":14})
ax.set_xlim(-100,100)
ax.set_ylim(-20,120)
ax.vlines(0,-20,120)
ax.text(-5, 10,'equator', rotation=90, fontdict={"fontsize":12})

ax.grid()
plt.show()
```


![png](../images/output_16_0.png)


## Latitude vs Cloudiness Plot


```python
fig,ax = plt.subplots(figsize=(10,6))
ax.scatter(weather_df['latitude'],weather_df['cloudiness'],
           c=weather_df['cloudiness'], cmap=plt.cm.binary, zorder=2)
ax.set_title('City Latitude vs Cloudiness ({})'.format(plot_date), fontdict={"fontsize":16})
ax.set_xlabel('Latitude', fontdict={"fontsize":14})
ax.set_ylabel('Cloudiness (%)', fontdict={"fontsize":14})
ax.set_xlim(-100,100)
ax.set_ylim(-20,120)
ax.vlines(0,-20,120)
ax.text(-5,113,'equator', rotation=90, fontdict={"fontsize":12})

ax.grid()
plt.show()
```


![png](../images/output_18_0.png)


## Latitude vs Wind Speed Plot


```python
fig,ax = plt.subplots(figsize=(10,6))
ax.scatter(weather_df['latitude'],weather_df['wind_speed'],
           c=weather_df['wind_speed'], cmap=plt.cm.winter, zorder=2)
ax.set_title('City Latitude vs Wind Speed ({})'.format(plot_date), fontdict={"fontsize":16})
ax.set_xlabel('Latitude', fontdict={"fontsize":14})
ax.set_ylabel('Wind Speed (mph)', fontdict={"fontsize":14})
ax.set_xlim(-100,100)
ax.set_ylim(-5,60)
ax.vlines(0,-5,60)
ax.text(-5,50,'equator', rotation=90, fontdict={"fontsize":12})

ax.grid()
plt.show()
```


![png](../images/output_20_0.png)


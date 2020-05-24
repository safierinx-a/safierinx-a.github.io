---
title: "Indian MSME Heatmap"
---

I was going through data.gov.in when I found a data set detailing the number ofManufacturing and Service based Micro, Small & Medium Enterprises (MSMEs) by district. I thought it would be a fun visualization to see these as a heatmap and maybe try to see if there are any trends in the data set. I found the API weird so I just downloaded it as a csv (March 27th, 2020)


```python
import pandas as pd
data = pd.read_excel(r"C:\Users\APOORV\AppData\Local\Temp\datafile.xls")
#data.sort_values(['state_id', 'DistrictName'])
data.head()

```

    WARNING *** OLE2 inconsistency: SSCS size is 0 but SSAT size is non-zero
    




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
      <th>StateName</th>
      <th>state_id</th>
      <th>DistrictName</th>
      <th>DISTRICT_ID</th>
      <th>TotalUAM</th>
      <th>General</th>
      <th>SC</th>
      <th>ST</th>
      <th>OBC</th>
      <th>_CreatedDate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>ANANTHAPUR</td>
      <td>520</td>
      <td>9601</td>
      <td>6585</td>
      <td>234</td>
      <td>102</td>
      <td>2680</td>
      <td>2020-03-26T08:20:24.580</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>CHITOOR</td>
      <td>511</td>
      <td>6689</td>
      <td>3808</td>
      <td>484</td>
      <td>62</td>
      <td>2335</td>
      <td>2020-03-26T08:20:24.580</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>EAST GODAVARI</td>
      <td>512</td>
      <td>11445</td>
      <td>5739</td>
      <td>1938</td>
      <td>89</td>
      <td>3679</td>
      <td>2020-03-26T08:20:24.580</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>GUNTUR</td>
      <td>509</td>
      <td>9226</td>
      <td>6311</td>
      <td>619</td>
      <td>357</td>
      <td>1939</td>
      <td>2020-03-26T08:20:24.580</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>KRISHNA</td>
      <td>517</td>
      <td>8201</td>
      <td>5687</td>
      <td>685</td>
      <td>65</td>
      <td>1764</td>
      <td>2020-03-26T08:20:24.580</td>
    </tr>
  </tbody>
</table>
</div>



Let's try to see them by state first. I just need to aggregate the data by state and we're good to go.


```python
df1 = data.groupby('StateName').agg(lambda x: x.tolist())
state = pd.DataFrame(columns=['name', 'manu', 'service'])
df1.head()
nam = []
man = []
ser = []
for i, j in df1.iterrows():
    mansum = 0
    servsum = 0
    for k in j[3]:
        mansum+=k
    for k in j[4]:
        servsum +=k
    state = state.append({'name': i, 'manu': mansum, 'service': servsum}, ignore_index=True)
    nam.append(i)
    man.append(mansum)
    ser.append(servsum)
state.head()

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
      <th>name</th>
      <th>manu</th>
      <th>service</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANDAMAN AND NICOBAR ISLANDS</td>
      <td>1892</td>
      <td>1543</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ANDHRA PRADESH</td>
      <td>119166</td>
      <td>55624</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARUNACHAL PRADESH</td>
      <td>950</td>
      <td>124</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ASSAM</td>
      <td>11065</td>
      <td>6781</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BIHAR</td>
      <td>305777</td>
      <td>73232</td>
    </tr>
  </tbody>
</table>
</div>




```python
state.to_csv("C:/Users/Apoorv/MSME state data.csv")
```

Let's plot these out on a simple bar graph


```python

import matplotlib.pyplot as plt; plt.rcdefaults()
import numpy as np
import matplotlib.pyplot as plt

w = .3
#plt.rcParams['font.family'] = 'sans-serif'
#plt.rcParams['font.sans-serif'] = 'Helvetica'
x = np.arange(len(nam))

plt.style.use('ggplot')
plt.rcParams['axes.edgecolor']='#333F4B'
plt.rcParams['axes.linewidth']=0.8
plt.rcParams['xtick.color']='#333F4B'
plt.rcParams['ytick.color']='#333F4B'
plt.figure(figsize = (20, 20), facecolor = None) 
manuf = plt.bar(x, man, color = 'b', align = 'center', width = w)
serv = plt.bar(x+w, ser, color = 'r', align = 'center', width = w)
plt.xticks(x + w /2, nam, rotation='vertical')
plt.legend([manuf, serv], ['Service MSMEs', 'Manufacturing MSMEs'])
plt.savefig("bar.jpg")

```


![png](output_6_0.png)


Maharashtra has a huge amount of MSMEs as compared to the rest of the country. Especially in the manufacturing sector. I wonder if there's a specific reason for this. Maybe I'll try to find funding details by state and see if there's a correlation later. Onto the heatmap for now.

I got the shapefile for India from gadm.org



```python
import numpy as np
import shapefile as shp
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
shp_path = r"C:\Users\APOORV\Downloads\gadm36_IND_shp\gadm36_IND_1.shp"
list = r"C:\Users\APOORV\Downloads\State long lat - Sheet1(2).csv"
import geopandas as gpd
pts = gpd.GeoDataFrame.from_file(shp_path)
pts.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x22180e8d710>




![png](output_10_1.png)


Waiting around is a pain so I'll add a loading bar to see the progress


```python
import geopy as gp
lats = []
longs = []
locator = gp.Nominatim(user_agent="apoorv")
from geopy.extra.rate_limiter import RateLimiter
r = []
for i, j in data.iterrows():
    k = j[2] + "," + j[0]
    r.append(k)
data['loc'] = r
geocode = RateLimiter(locator.geocode, min_delay_seconds=1)
from tqdm import tqdm
tqdm.pandas()
data['location'] = data['loc'].progress_apply(geocode)
data['location'] = data['loc'].apply(geocode)

data['point'] = data['location'].apply(lambda loc: tuple(loc.point) if loc else None)
# 4 - split point column into latitude, longitude and altitude columns
data[['latitude', 'longitude', 'altitude']] = pd.DataFrame(data['point'].tolist(), index=data.index)
```


```python
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
      <th>StateName</th>
      <th>state_id</th>
      <th>DistrictName</th>
      <th>DISTRICT_ID</th>
      <th>TotalManufacturingUnit</th>
      <th>TotalServiceUnit</th>
      <th>_CreatedDate</th>
      <th>loc</th>
      <th>location</th>
      <th>point</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>altitude</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>ANANTHAPUR</td>
      <td>520</td>
      <td>9601</td>
      <td>8506</td>
      <td>2020-03-26T08:35:25.493</td>
      <td>ANANTHAPUR,ANDHRA PRADESH</td>
      <td>(Ananthapur-Dharmavaram Road, Pesaragunta, Rap...</td>
      <td>(14.5406722, 77.6619216, 0.0)</td>
      <td>14.540672</td>
      <td>77.661922</td>
      <td>0.0</td>
      <td>18107</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>CHITOOR</td>
      <td>511</td>
      <td>6689</td>
      <td>16391</td>
      <td>2020-03-26T08:35:25.493</td>
      <td>CHITOOR,ANDHRA PRADESH</td>
      <td>(Chitoor Math, West Mada Street, Srivari Templ...</td>
      <td>(13.6860791, 79.3451226, 0.0)</td>
      <td>13.686079</td>
      <td>79.345123</td>
      <td>0.0</td>
      <td>23080</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>EAST GODAVARI</td>
      <td>512</td>
      <td>11445</td>
      <td>10167</td>
      <td>2020-03-26T08:35:25.493</td>
      <td>EAST GODAVARI,ANDHRA PRADESH</td>
      <td>(East Godavari, Andhra Pradesh, India, (17.233...</td>
      <td>(17.233496, 81.7225986, 0.0)</td>
      <td>17.233496</td>
      <td>81.722599</td>
      <td>0.0</td>
      <td>21612</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>GUNTUR</td>
      <td>509</td>
      <td>9226</td>
      <td>11694</td>
      <td>2020-03-26T08:35:25.493</td>
      <td>GUNTUR,ANDHRA PRADESH</td>
      <td>(Guntur, Andhra Pradesh, 522001, India, (16.29...</td>
      <td>(16.2915189, 80.4541588, 0.0)</td>
      <td>16.291519</td>
      <td>80.454159</td>
      <td>0.0</td>
      <td>20920</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ANDHRA PRADESH</td>
      <td>28</td>
      <td>KRISHNA</td>
      <td>517</td>
      <td>8201</td>
      <td>10480</td>
      <td>2020-03-26T08:35:25.493</td>
      <td>KRISHNA,ANDHRA PRADESH</td>
      <td>(Krishna, Andhra Pradesh, India, (16.6691525, ...</td>
      <td>(16.6691525, 80.7190024, 0.0)</td>
      <td>16.669152</td>
      <td>80.719002</td>
      <td>0.0</td>
      <td>18681</td>
    </tr>
  </tbody>
</table>
</div>



Gotta manually fix the NaN values


```python
rows_with_nan = [index for index, row in data.iterrows() if row.isnull().any()]
```


```python
data.at[698, 'latitude'] = 10.5667
data.at[698, 'longitude'] = 72.6417
data.at[45, ['latitude', 'longitude', 'altitude']] = [26.183333, 91.733333, 0]
data.at[46, ['latitude', 'longitude', 'altitude']] = [26.183333, 93.566667, 0]
data.at[96, ['latitude', 'longitude', 'altitude']] = [21.67, 82.17, 0]
data.at[118, ['latitude', 'longitude', 'altitude']] = [21.1, 81.03, 0]
data.at[127, ['latitude', 'longitude', 'altitude']] = [24.0283, 73.0414, 0]
data.at[132, ['latitude', 'longitude', 'altitude']] = [22.305556, 74.013889, 0]
data.at[134, ['latitude', 'longitude', 'altitude']] = [22.2, 69.65, 0]
data.at[135, ['latitude', 'longitude', 'altitude']] = [22.833889, 74.257778, 0]
data.at[224, ['latitude', 'longitude', 'altitude']] = [12.87, 74.88, 0]
data.at[241, ['latitude', 'longitude', 'altitude']] = [14.6, 74.7, 0]
data.at[254, ['latitude', 'longitude', 'altitude']] = [9.27, 76.78, 0]
data.at[328, ['latitude', 'longitude', 'altitude']] = [219.16, 77.32, 0]
data.at[374, ['latitude', 'longitude', 'altitude']] = [25.9, 94.783333, 0]
data.at[420, ['latitude', 'longitude', 'altitude']] = [30.94, 74.62, 0]
data.at[433, ['latitude', 'longitude', 'altitude']] = [31.125278, 76.116389, 0]
data.at[493, ['latitude', 'longitude', 'altitude']] = [11.4916, 76.7337, 0]

data.at[515, ['latitude', 'longitude', 'altitude']] = [19.3619, 79.2930, 0]
data.at[527, ['latitude', 'longitude', 'altitude']] = [17.3891, 77.8367, 0]
data.at[547, ['latitude', 'longitude', 'altitude']] = [26.4684, 82.6915, 0]
data.at[610, ['latitude', 'longitude', 'altitude']] = [26.7672, 83.0361, 0]
data.at[614, ['latitude', 'longitude', 'altitude']] = [27.2991, 83.0928, 0]
data.at[629, ['latitude', 'longitude', 'altitude']] = [30.2844, 78.9811, 0]
data.at[636, ['latitude', 'longitude', 'altitude']] = [26.3452, 89.4482, 0]
data.at[645, ['latitude', 'longitude', 'altitude']] = [22.5726, 88.3639, 0]
data.at[649, ['latitude', 'longitude', 'altitude']] = [22.6168, 88.4029, 0]
data.at[653, ['latitude', 'longitude', 'altitude']] = [22.1352, 88.4016, 0]
data.at[656, ['latitude', 'longitude', 'altitude']] = [7.1205, 93.7842, 0]
data.at[658, ['latitude', 'longitude', 'altitude']] = [11.7401, 92.6586, 0]
data.at[660, ['latitude', 'longitude', 'altitude']] = [20.1809, 73.0169, 0]




```

Let's make a heatmap! We'll do the Manufacturing one first.


```python
import folium
from folium.plugins import HeatMap



max_amount = float(data['TotalServiceUnit'].max())

hmap = folium.Map(location=[22, 72.5], zoom_start=3, ) 

hm_wide = HeatMap(data[['latitude','longitude','TotalServiceUnit']].values,
                   min_opacity=0.2,
                   max_val=max_amount,
                   radius=17, blur=15, 
                   max_zoom=1, 
                 )

hmap.add_child(hm_wide)
hmap.save('Service_HeatMap.html')
```


```python

max_amount = float(data['TotalManufacturingUnit'].max())

hmap = folium.Map(location=[22, 72.5], zoom_start=3, ) 

hm_wide = HeatMap(data[['latitude','longitude','TotalManufacturingUnit']].values,
                   min_opacity=0.2,
                   max_val=max_amount,
                   radius=17, blur=15, 
                   max_zoom=1, 
                 )

hmap.add_child(hm_wide)
hmap.save('Manufacturing_HeatMap.html')
```

Let's make one for the total too now that we're at it.


```python
data['total'] = data['TotalServiceUnit'] + data['TotalManufacturingUnit']
max_amount = float(data['total'].max())

hmap = folium.Map(location=[22, 72.5], zoom_start=3, ) 

hm_wide = HeatMap(data[['latitude','longitude','total']].values,
                   min_opacity=0.2,
                   max_val=max_amount,
                   radius=17, blur=15, 
                   max_zoom=1, 
                 )

hmap.add_child(hm_wide)

```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF9mYjUwN2YzNTBjY2Y0ODU3OTY5NzU1ZjI4NDk4MDQwZiB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9sZWFmbGV0LmdpdGh1Yi5pby9MZWFmbGV0LmhlYXQvZGlzdC9sZWFmbGV0LWhlYXQuanMiPjwvc2NyaXB0Pgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2ZiNTA3ZjM1MGNjZjQ4NTc5Njk3NTVmMjg0OTgwNDBmIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF9mYjUwN2YzNTBjY2Y0ODU3OTY5NzU1ZjI4NDk4MDQwZiA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF9mYjUwN2YzNTBjY2Y0ODU3OTY5NzU1ZjI4NDk4MDQwZiIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMjIuMCwgNzIuNV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAzLAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMjYwZmMzNmFlNDkxNGE5YmEwMWNlOTkxNWFkZDJiM2EgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIkRhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiNTA3ZjM1MGNjZjQ4NTc5Njk3NTVmMjg0OTgwNDBmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgaGVhdF9tYXBfMmY4MjdkNmZiZjZlNDAxNGIzMjA5OTQzMTAwMzljYTMgPSBMLmhlYXRMYXllcigKICAgICAgICAgICAgICAgIFtbMTQuNTQwNjcyMiwgNzcuNjYxOTIxNiwgMTgxMDcuMF0sIFsxMy42ODYwNzkxLCA3OS4zNDUxMjI2LCAyMzA4MC4wXSwgWzE3LjIzMzQ5NiwgODEuNzIyNTk4NiwgMjE2MTIuMF0sIFsxNi4yOTE1MTg5LCA4MC40NTQxNTg4LCAyMDkyMC4wXSwgWzE2LjY2OTE1MjUsIDgwLjcxOTAwMjQsIDE4NjgxLjBdLCBbMTUuODMwOTI1MSwgNzguMDQyNTM3MywgMTMyMjUuMF0sIFsxNS41LCA3OS41LCAzOTg3OS4wXSwgWzE0LjI5Mzk4MTgwMDAwMDAwMSwgNzkuNzc5NDM4NDI4MDUwMzYsIDUxNzE1LjBdLCBbMTguMzIwMDIyMDUsIDgzLjkxNjA3NzE5OTM3MTY2LCA5NzYwLjBdLCBbMTcuNzIzMTI3NiwgODMuMzAxMjg0MiwgMjQzNjkuMF0sIFsxOC4xMTIwODE5LCA4My40MDUyMTk2MjI0ODg4LCAyOTE5NS4wXSwgWzE3LjAsIDgxLjE2NjY2NywgMzA0MDYuMF0sIFsxNS45MjQwOTA1LCA4MC4xODYzODA5LCAzNjIxMS4wXSwgWzI4LjExMTQzMDEsIDk2LjgyNjk5OTYzMzk4MzczLCAxMi4wXSwgWzI3LjEzOTg0MDIsIDk1LjczODI0ODUsIDE0OC4wXSwgWzI4LjcsIDk1LjcsIDE0LjBdLCBbMjcuMywgOTMuMDUsIDE3LjBdLCBbMjguMywgOTUuMTUsIDEwNS4wXSwgWzI3Ljk4NjM2NzY1LCA5My4xNTc4MzQ2ODY0NDEwMiwgMjAuMF0sIFsyOC4wMzUwMTYxNDk5OTk5OTcsIDk2LjE3MTM1MTU0MjIwNjc2LCA5MS4wXSwgWzI2Ljg3OTQ1NDYwMDAwMDAwMywgOTUuMzE1NTY1MjQ1Njk0MSwgMTguMF0sIFsyOC4yMTQ5NTYyNSwgOTUuODc4Nzc0MDM1NjcwMTksIDU2LjBdLCBbMjcuOCwgOTMuNiwgMTI0LjBdLCBbMjcuMjg2MzY2NjQ5OTk5OTk4LCA5My42MTg3NTE2ODIwNjM4MiwgNjQwLjBdLCBbMjcuNjM4OTg5NDUsIDkxLjgxODkyNDgzNzQ4ODI3LCAxMC4wXSwgWzI3LjAzMDE2ODk1LCA5NS40OTc5MTc1NDUwNTU4MywgNy4wXSwgWzI4LjczMTIwNDU1LCA5NS4wMzc3MzE1MTM1MDk0NiwgMTAuMF0sIFsyOC4zLCA5NC4wLCA5NS4wXSwgWzI3LjQsIDkyLjM1LCA0OS4wXSwgWzI4LjQsIDk0LjU1LCAxMzMuMF0sIFsyNi42Mjc4MTgyNSwgOTEuNDYxNDg1NzgwNTczMzMsIDM2Mi4wXSwgWzI2LjM0MTQyMjE1LCA5MS4wMTY3NDMyODQ3NDA3MywgODgyLjBdLCBbMjYuNDgwMDEyNiwgOTAuNTU4MDM4NywgMzc5LjBdLCBbMjQuNzU4NjM5OTUwMDAwMDAzLCA5Mi44ODE2NjQ3NTc2ODIxOSwgNzM4LjBdLCBbMjYuNjQ2NDIzMSwgOTAuNjI0MTU2OTg3Mjk4NzUsIDM5OS4wXSwgWzI2Ljc1LCA5Mi41LCA2NTQuMF0sIFsyNy40NjY4MzE0OTk5OTk5OTgsIDk0LjUxNzg2NTA4NzA4ODUyLCAyOTQuMF0sIFsyNi4wMTg5NDYyNSwgODkuOTQxODg5NTI0NjE1MjIsIDYwMC4wXSwgWzI3LjQ4NDQ1OTcsIDk0LjkwMTk0NDcsIDk2NS4wXSwgWzI1LjQwMDM2MjU0OTk5OTk5NywgOTMuMDc1MjI5MjI2Nzg5MDUsIDcwLjBdLCBbMjYuMTc4MzYyMywgOTAuNjIyNjY4MywgMzQ4LjBdLCBbMjYuNDA0OTAwOTUsIDk0LjAzMjA5MTEyMTIwNjI2LCA3ODAuMF0sIFsyNC42ODg0NTM2NSwgOTIuNTczODE1MjgyNDA5NDQsIDE0Ni4wXSwgWzI2Ljc1Nzc5MjUsIDk0LjIwNzk2NDUsIDQyOS4wXSwgWzI2LjEyNzU1NTE1LCA5MS4yOTk4NzE0Nzk3MzU3LCAxNTQ3LjBdLCBbMjYuMTgzMzMzLCA5MS43MzMzMzMsIDQwMTMuMF0sIFsyNi4xODMzMzMsIDkzLjU2NjY2NywgMjcyLjBdLCBbMjQuODQ4MzQ4MiwgOTIuMzMyMjEyOTIzMTAzMjgsIDIyNy4wXSwgWzI2LjU4Njg3Njk1LCA5MC4yOTI5MzEwNTEyNDc5NiwgMzY4LjBdLCBbMjcuMDgyNzE1OCwgOTMuOTk3ODQyMywgNDI1LjBdLCBbMjYuMjkwNzEyLCA5Mi4yOTIyMTI1Njc2NTIyNiwgMzM2LjBdLCBbMjYuMzA0MTQ5MTUsIDkyLjcxNjA1OTc0NDMwMzQ2LCAxNjQwLjBdLCBbMjYuNDIzODYxNjUsIDkxLjQ1MTIyNzU0NjIxMTcsIDQyMi4wXSwgWzI2Ljk4MzQ5MzYsIDk0LjYzOTQyMjYsIDU0Ni4wXSwgWzI2Ljc2Nzc3MDc0OTk5OTk5NywgOTIuNzAxNDA4ODU4NTA1MzIsIDg2Ni4wXSwgWzI3LjUyMDE2NTc1LCA5NS41MzA2MzQ3MzEyODc5LCA4OTMuMF0sIFsyNi43NzY0ODAzMDAwMDAwMDMsIDkyLjEwNTM3NTMzODk3MTU2LCAyOTAuMF0sIFsyNi4yNjQ5ODc5NSwgODcuMzcxNjI2NDgxMDY3OTgsIDQ2NjkzLjBdLCBbMjUuMTgwNzc3MTQ5OTk5OTk3LCA4NC42NTkyMDIwMzAyMTU1NCwgMjY1MS4wXSwgWzI0Ljc1LCA4NC41LCA3OTk5LjBdLCBbMjQuODY1NTY4MSwgODYuODcyNzg2NDIyOTU1OTcsIDI3ODQzLjBdLCBbMjUuNTEyNzE5NCwgODYuMDkwNTcxMDkxNDAwMjEsIDI5NTkyLjBdLCBbMjUuMjg2Njk4MDUsIDg3LjEzMjI1MzU1MTExMTQ4LCAyNTI3Ni4wXSwgWzI1LjI1LCA4NC4yNSwgOTgyNy4wXSwgWzI1LjU2MjA3MTUwMDAwMDAwMiwgODQuMDE1NjcyMDQ2NTEyODksIDg3NTguMF0sIFsyNi4wODMxNDMyNSwgODYuMDMyNTcwOTY2MTI4MjIsIDM3MDM4LjBdLCBbMjQuNzk2NDM1NSwgODUuMDA3OTU2MywgMjE0MzkuMF0sIFsyNi40MjA3MjY1LCA4NC4zNzQwNjQwNzQxNTE2NSwgOTI4Ni4wXSwgWzI0Ljk2Nzg1MjA0OTk5OTk5OCwgODYuMTczNTExNDcyODkyODMsIDE1ODI4LjBdLCBbMjUuMTUyNDcxMzUsIDg1LjAwNjg3ODQ4OTgyMTkxLCAzNzI2LjBdLCBbMjQuOTc0ODY0MzQ5OTk5OTk3LCA4My41NzY0MzkwNDcwMDczMSwgNjkyMS4wXSwgWzI1LjU2MDkwMDM0OTk5OTk5NywgODcuNjQ3NjUzNTM5OTcxNDksIDE5OTYyLjBdLCBbMjUuNDkxNjg2OCwgODYuNjk5MTA4NDgyMTIyMTcsIDEyODE4LjBdLCBbMjYuMjk4NjM3OTUsIDg3Ljk1MzE0ODEwOTIzNDA5LCAxNDgwNi4wXSwgWzI1LjE2NzcwMjkwMDAwMDAwMiwgODYuMDU5MzU5MjQzMDQwMTgsIDE0OTE5LjBdLCBbMjUuOTI1OTU2MiwgODYuODIwNDI5NjEwMzU2NzcsIDgxNzYuMF0sIFsyNi4zNTAwNzE5OTk5OTk5OTcsIDg2LjIyOTE0Njg4OTYyMjcxLCAzNjQ4Mi4wXSwgWzI1LjIyMDgxMTY1LCA4Ni41MTcyMDM2NzIzNTI4NSwgOTYyNC4wXSwgWzI2LjE0ODY1ODEsIDg1LjM0MDAxMjgyNDc5NjI2LCA2ODE2Ni4wXSwgWzI1LjEzNjQ5MTQsIDg1LjQ0MzY1NDY0ODU1NjgyLCAyMzgwMS4wXSwgWzI0LjgxODEyMzk5OTk5OTk5NywgODUuNTE4NTU2MjMzNjkyMSwgMTIwNzEuMF0sIFsyNy4wLCA4NC41LCA1MzkzNS4wXSwgWzI1LjYwOTMyMzksIDg1LjEyMzUyNTIsIDYyMjUyLjBdLCBbMjYuNjQ2NTk2OSwgODQuOTA5MTU1MiwgNDc3NzcuMF0sIFsyNS43NzczNjAxLCA4Ny40NzMxMjI3LCAzNjkxMi4wXSwgWzI0LjkzOTI4Nzc1LCA4NC4wMTk0NTQ0ODg1MjA2NywgMjg4NTEuMF0sIFsyNS44MzI2NDE2NSwgODYuNjE0ODkyODMzNDI4MTQsIDE4MDk0LjBdLCBbMjUuNzc0NDkyOSwgODUuODY2MjY4MTMzNzAxMjMsIDQxNTQwLjBdLCBbMjUuOTA4MzU0OTUsIDg0Ljc4MDc0NzYwNTk0MzIzLCAyNTEyMC4wXSwgWzI1LjEyNjM1NDI1LCA4NS44MTUxMDYyMDk0NDE5NCwgNTk3Ni4wXSwgWzI2LjQ5NDM2OTQ1LCA4NS4yNzI4MDU1OTUxMzk5OSwgNTU2OC4wXSwgWzI2LjU4NzY1NDcsIDg1LjUwNTY3NDQsIDM4NTQ4LjBdLCBbMjYuMTMxMDA0MywgODQuMzkxMjU2NjA1OTUxOTgsIDc0NDguMF0sIFsyNi4yNzY4MTE4LCA4Ni43OTY1ODY2MjYxNDgxMSwgMTMxODkuMF0sIFsyNS43NSwgODUuNDE2NjY3LCAzNzk0Ni4wXSwgWzIwLjczMjE4MzY1LCA4MS4xNDMzNTczODM2NTA0MywgMzAyNS4wXSwgWzIxLjY3LCA4Mi4xNywgMzUzOC4wXSwgWzIzLjYyNjMwMTYsIDgzLjQ4MjQ1MDkxNjU2NTI3LCA0NDEuMF0sIFsxOS4xMTkxMjgyNSwgODEuODI5MTg2NDg5NzE3MTMsIDEwNzYuMF0sIFsyMS42NzgzOTY5LCA4MS40MzA0NzYyMjgzMzMzMywgMTM1Mi4wXSwgWzE4Ljc2OTA0MTM1MDAwMDAwMiwgODAuNzQxNTExOTI5NjYyMzEsIDU3LjBdLCBbMjIuMzgzMzMzLCA4Mi4xMzMzMzMsIDY4MDQuMF0sIFsxOC44NjQwNjQ4LCA4MS4zODMzOTQ2ODczODY0OCwgMzEzLjBdLCBbMjAuNTM3Mjc2NjUwMDAwMDAzLCA4MS43MDkxNTE0NTczNzY2NiwgMjU3NS4wXSwgWzIxLjE5OTAzNTM1LCA4MS4zOTc5NTQ1NTczNjU3LCA5MDU0LjBdLCBbMjAuNDI4NTcwMTUsIDgyLjI0Njk4OTg0ODE3NDM3LCAxMDY2LjBdLCBbMjEuOTY3MTI2Mzk5OTk5OTk4LCA4Mi42NjQ3MjQyNDk2MTA0LCAzMjc0LjBdLCBbMjIuNzY2NzgzNTUsIDgzLjk1MzA0NTE4OTUzMzE3LCA0MDUuMF0sIFsyMi4xMTQyODg2NSwgODEuMjUxNTc5ODg3NTE5NTEsIDExNDkuMF0sIFsyMC4xMjcwMDI2LCA4MC45ODEwOTM0OTQ3MjkxMSwgNzQ2LjBdLCBbMTkuNzA0MzY0NjUsIDgxLjc0NjU1MDIxMTI4NzgzLCA3MDkuMF0sIFsyMi41MTk3Njk1NSwgODIuNjI5NTE0NjI0MTY4MTgsIDIzNTYuMF0sIFsyMy4yMzgyOTk5LCA4Mi4zOTA2NTg4LCA4OTguMF0sIFsyMS4xODgyMzMyLCA4Mi40ODQ4NDc2MjQ5OTk5OSwgMTgxOS4wXSwgWzIyLjI0NDUxNjI1LCA4MS42NTQwMTA5MzgxNzQ5OSwgNjUzLjBdLCBbMTkuNTg0MjAzMiwgODEuMTEzNjk4NTUxMjAyOSwgNzAuMF0sIFsyMi41LCA4My41LCAyNDgzLjBdLCBbMjEuMjM3OTQ2OSwgODEuNjMzNjgzMywgMTUwODUuMF0sIFsyMS4xLCA4MS4wMywgMzc0MC4wXSwgWzE4LjI2NjU0MjI1LCA4MS4yOTI1MjM0MTQ3NTA2NSwgNTYuMF0sIFsyMy4zNTE1MTUxNSwgODIuOTcyOTAyNzI2NjUwMTUsIDY2Mi4wXSwgWzIyLjk0OTQxODM1LCA4My4xNjU1NDkxNDU3OTQ5LCAxMTcxLjBdLCBbMTUuNjA0NDQxMjUsIDc0LjAwMTcyNDA2MzA3NDc1LCA0NjQ3LjBdLCBbMTUuMjE5NjEzNjUsIDc0LjExNTI4MTMwMTkxOTYsIDM1OTEuMF0sIFsyMy4wMjE2MjM4LCA3Mi41Nzk3MDY4LCAxOTE2NDUuMF0sIFsyMC44NjY2NjcsIDcwLjc1LCAzMzEyLjBdLCBbMjIuNTU4NDk5NSwgNzIuOTYyNTYyOSwgMTA2OTEuMF0sIFsyNC4wMjgzLCA3My4wNDE0LCA5NzMuMF0sIFsyNC4yNSwgNzIuNSwgNDc1Mi4wXSwgWzIxLjc1LCA3My4wLCAxMzExNy4wXSwgWzIxLjc3MTg4MzYsIDcyLjE0MTY0NDksIDE5MzEyLjBdLCBbMjIuMTY4NiwgNzEuNjY4NSwgMTQwNS4wXSwgWzIyLjMwNTU1NiwgNzQuMDEzODg5LCAxMDQwLjBdLCBbMjEuNTE3NDEwNCwgNzAuNDY0Mjc1NCwgNjUuMF0sIFsyMi4yLCA2OS42NSwgMTE0NC4wXSwgWzIyLjgzMzg4OSwgNzQuMjU3Nzc4LCAxMTgxLjBdLCBbMjMuMjIzMjg3NywgNzIuNjQ5MjI2NywgMTMzMjcuMF0sIFsyMC45OTExMDQ5LCA3MC42ODgwNjQwNTA0Njg0OCwgMjE5NS4wXSwgWzIyLjQ3MzI0MTUsIDcwLjA1NTIxMDIsIDExNjQ2LjBdLCBbMjEuNTE4NTU4OTAwMDAwMDAyLCA3MC40NTkxNzIxMTcyNDM3OCwgNjAzOC4wXSwgWzIzLjU4MzMzMywgNzAuMCwgMTExNzkuMF0sIFsyMi43NSwgNzIuODMzMzMzLCA0MDIwLjBdLCBbMjMuNjY2NjY3LCA3Mi41LCA2Nzk0LjBdLCBbMjMuMTkxNjk2NywgNzMuNjYyNjM1Mjk2NDAxOSwgNDIzLjBdLCBbMjIuODE3NjY2MiwgNzAuODM0NTkyOCwgNzI2NC4wXSwgWzIxLjczODMwMzk1MDAwMDAwMiwgNzMuNjI3MzYwMTU5MjEzMjgsIDQ2MS4wXSwgWzIwLjk1MjQwNywgNzIuOTMyMzgzMSwgNjUzOS4wXSwgWzIzLjAsIDc0LjAsIDI4MjUuMF0sIFsyMy43NzQwNTczNSwgNzEuNjgzNzM0NjU2Njg3MzQsIDIxMjUuMF0sIFsyMS42NDA5LCA2OS42MTEsIDE3MTkuMF0sIFsyMi4zMDUxOTkxLCA3MC44MDI4MzM1LCA4MjY2OC4wXSwgWzIzLjUsIDczLjI1LCAyNTk0LjBdLCBbMjEuMTg2NDYwNywgNzIuODA4MTI4MSwgMzEyMDI0LjBdLCBbMjIuNzUsIDcxLjY2NjY2NywgNTA3NS4wXSwgWzIxLjE5MDkxNDEsIDczLjU1NTQxMDg1NzQ1MTg2LCAxMjU2LjBdLCBbMjIuMjk3MzE0MiwgNzMuMTk0MjU2NywgNTA4MDQuMF0sIFsyMC40MzI0MDE4LCA3My4xNDExNzE1NTUzNTc3LCAxNTk3NS4wXSwgWzMwLjM4NDM2NzQsIDc2Ljc3MDQyMSwgNzg0Mi4wXSwgWzI4Ljc5MzE3MDMsIDc2LjEzOTEyODMsIDM0MzQuMF0sIFsyOC40MDI4MzcsIDc3LjMwODU2MjYsIDI0MzQ0LjBdLCBbMjkuNDY4ODMwMDk5OTk5OTk4LCA3NS41MjEwNzE0NjYzMTc0NywgNDc1MS4wXSwgWzI4LjQ2NDYxNDgsIDc3LjAyOTkxOTQsIDIzMTAyLjBdLCBbMjkuMTY4ODA3LCA3NS43NDYxMTAzLCA5MDI5LjBdLCBbMjguNTMzNjg0NCwgNzYuNjg5ODEyMTIwMzU2NDgsIDQwNzEuMF0sIFsyOS41LCA3Ni4yNSwgNjg2OS4wXSwgWzI5Ljc0NDc3MDc1LCA3Ni4zNTA5Mjc3MjI4NDc4NCwgODIyNi4wXSwgWzI5LjY4MDMyNjYsIDc2Ljk4OTYyNTQsIDEzNzc2LjBdLCBbMjkuOTY5Mzc0NywgNzYuODQ4Mjc4NywgNTgwNS4wXSwgWzI4LjI1LCA3Ni4xNjY2NjcsIDIxMzUuMF0sIFsyOC4wOTczNDUxNTAwMDAwMDIsIDc3LjA1MDg1NTgyMTAxODUyLCAxNzIxLjBdLCBbMjguMTI1MDI1NzUsIDc3LjM1ODMxMzAwNzczMDQ2LCAyNjMwLjBdLCBbMzAuNjE2MjE2NDUwMDAwMDAzLCA3Ny4wNDE5NzgwNDMyMTg3NSwgNDc2Ni4wXSwgWzI5LjM5MTI3NTMsIDc2Ljk3NzE2NzUsIDE3Mzk3LjBdLCBbMjguMTk1NjQ2OCwgNzYuNjE2NTE3OSwgMzc2NC4wXSwgWzI4LjkwMTA4OTksIDc2LjU4MDE5MzUsIDUwNTUuMF0sIFsyOS41ODMzMzMsIDc1LjA4MzMzMywgNzQzNi4wXSwgWzI5LjAwMzMxNDQsIDc3LjAxNjczMjMsIDEwMjI1LjBdLCBbMzAuMjExMjAwMywgNzcuMjg2Mzg5NzIzNjA4OTUsIDgxMzMuMF0sIFszMS4zMzgwOTU5LCA3Ni43NjExNjMxLCAzMTUuMF0sIFszMi41LCA3Ni41LCA2NDEuMF0sIFszMS43NDkxMzMzLCA3Ni41MDM2ODMyLCA0NDkuMF0sIFszMi4xNjY2NjcsIDc2LjI1LCAyMTc2LjBdLCBbMzEuNTg2MzQ2LCA3OC4zOTY4NDQxLCAxMjUuMF0sIFszMi4wMDE4NjMyNSwgNzcuMzc4OTk2Mzk3NDEzMzIsIDkyMy4wXSwgWzMyLjUwMzg0OTUsIDc3LjU4MjI3MTg4MjM2NDI4LCAzMS4wXSwgWzMxLjY3MjMzOTIsIDc2Ljk1MzMxNjkwNzc5MTQ1LCAxMjQ4LjBdLCBbMzEuMTA0MTUyNiwgNzcuMTcwOTcyOSwgMTQxNy4wXSwgWzMwLjc1LCA3Ny41LCA4MDYuMF0sIFszMC45MjU4OTU4NSwgNzcuMDgyMDA1MDkwNjM2MjQsIDQwOTUuMF0sIFszMS41ODMzMzMsIDc2LjI1LCAxNTQ0LjBdLCBbMjMuNjk5MTI3OTQ5OTk5OTk4LCA4NS45OTEwNjg5NDE2NTAyMSwgMTMzNzEuMF0sIFsyNC4yMDE2MjQ5NSwgODQuODc4MTc2OTE5NDE3MzMsIDIwMDAuMF0sIFsyNC40NzY2NDIzLCA4Ni42MDY3MzI0NTM4Njk0NSwgNDE5NC4wXSwgWzIzLjc5NTI4MDksIDg2LjQzMDk2MzgsIDEzMjU3LjBdLCBbMjQuMjUzODUxMiwgODcuMzAwNjQ3MTQxOTIyMjQsIDI4NjEuMF0sIFsyMi41LCA4NS41LCAyNDgyOC4wXSwgWzI0LjE1NTI5MDM1LCA4My44MjUyMjU4NDQwODM0OSwgODM2OC4wXSwgWzI0LjI1LCA4NS45MTY2NjcsIDEwNTA5LjBdLCBbMjQuNzQzMzQzMDUsIDg3LjI5Mjg1MDYxNzk2ODU4LCA0MzIxLjBdLCBbMjMuMDkxNjc0MSwgODQuNTczODQ4MDcxOTgwMiwgMTk3NC4wXSwgWzI0LjAsIDg1LjI1LCAxMDQzMi4wXSwgWzIzLjkyMzI4NjcsIDg2Ljc3MzUzNDkzODkxNDU4LCA4MjMuMF0sIFsyMy4xMDAxODEwNSwgODUuMzQ1NDU5MzYxNzk4NSwgMjUyMS4wXSwgWzI0LjQ5MzEyMzksIDg1LjU1NjA4MTA3MjAzODgxLCAyMzM0LjBdLCBbMjMuNzQxODY1NDUsIDg0LjUxMzAxMzY2NTIyMTA1LCA2MDIuMF0sIFsyMy40NjkyMDA1NSwgODQuNzM1Njg0MjMwNzE3NTEsIDIwNDEuMF0sIFsyNC42Mzc5MDY0LCA4Ny44NTU2OTYyLCAxNjQ4LjBdLCBbMjMuOTE2NjY3LCA4NC4wODMzMzMsIDQyNTYuMF0sIFsyMy41OTg3NzU5NDk5OTk5OTcsIDg1LjUzNjkxNTY0NjYzMTI3LCA1MjQ2LjBdLCBbMjMuMzcwMDM1NCwgODUuMzI1MDEzMiwgMjE5NzkuMF0sIFsyNS4yMzYxMTk2LCA4Ny42MzQzMDY0LCAxODk2LjBdLCBbMjIuODIzMzY5MywgODUuOTE5NjUzOTkwMjk5NDIsIDc0MzUuMF0sIFsyMi42NjY0MzI1LCA4NC41NTAyNjc5MTE5MjIxLCA0NjkuMF0sIFsyMi43ODU5MjMyLCA4Ni4xNzE2MDkzLCAyMjIwLjBdLCBbMTYuMTg1MzE2NiwgNzUuNjk2NzkxOSwgNjc5Mi4wXSwgWzE1LjI0ODU0MTMsIDc2LjgzMjM2NDgsIDQwMzAuMF0sIFsxNS44NTcyNjY2LCA3NC41MDY5MzQzLCAzODU5Ny4wXSwgWzEzLjE4MDc3ODI1LCA3Ny4zNDE3NDIzNjkyMzM3MSwgMTY2NzkuMF0sIFsxMi45NDUxNDIyNSwgNzcuNTUzNjQ0OTk5NzExMjgsIDEzMjY4NC4wXSwgWzE4LjA4MzMzMywgNzcuMzMzMzMzLCAzOTM1LjBdLCBbMTEuODA2NTU3OCwgNzYuNjkwODg0OSwgMTA1Ni4wXSwgWzEzLjQyNzQ0MDUsIDc3LjczMjE0MzMsIDIxMTIuMF0sIFsxMy4zMTgwMTQsIDc1Ljc3Mzg3NDMsIDMwMjQuMF0sIFsxNC4yMjY2NDQzLCA3Ni40MDA1MTIyLCAyMzYxLjBdLCBbMTIuODcsIDc0Ljg4LCAxMjc2OS4wXSwgWzE0LjQ2NjQ0NTcwMDAwMDAwMSwgNzUuOTE5NzUwNzkwOTM0MywgNDM3NS4wXSwgWzE1LjQ1NDA1MDUsIDc1LjAwNjY1MTYsIDE5NDE3LjBdLCBbMTUuNDI2MzY0NywgNzUuNjMwMDc4NiwgMzA3OS4wXSwgWzEzLjAwNzA4MTcsIDc2LjA5OTI3MDMsIDMyNDUuMF0sIFsxNC43ODc0ODI1LCA3NS4zOTk2NzMxLCAyODgxLjBdLCBbMTcuMTY2NjY3LCA3Ny4wODMzMzMsIDM1NjguMF0sIFsxMi4zODIyODA4LCA3NS42NjUyMzUzMzU4NjIwOCwgMTM2MC4wXSwgWzEzLjEzNjk5OTYsIDc4LjEzMzk2MDYsIDI4MjguMF0sIFsxNS4zNDg0MTQ0LCA3Ni4xNTQ3NDIxLCAxNDc3LjBdLCBbMTIuNTIzODg4OCwgNzYuODk2MTk2MSwgMTg2MS4wXSwgWzEyLjMwNTE4MjgsIDc2LjY1NTM2MDksIDEyMzg3LjBdLCBbMTYuMDgzMzMzLCA3Ny4xNjY2NjcsIDI3NDEuMF0sIFsxNS40MjIwNDcsIDc0LjQ5MDA4NjMsIDMyMDUuMF0sIFsxMy45MzExNzIzLCA3NS41Njk3MzcsIDQ2NzYuMF0sIFsxMy4zNDAwNzcxLCA3Ny4xMDA2MjA4LCA2NTIzLjBdLCBbMTMuMzQxOTE2OSwgNzQuNzQ3MzIzMiwgNjIyNi4wXSwgWzE0LjYsIDc0LjcsIDUxNTAuMF0sIFsxNi42NjY2NjcsIDc1LjkxNjY2NywgNjI1NS4wXSwgWzE2Ljc2ODkwNDQsIDc3LjEzODAzODEsIDEyOTcuMF0sIFs5LjQ4ODcwMDU1LCA3Ni40MTI1NjQxMDk2OTYyNiwgODQ0Mi4wXSwgWzEwLjAzODM5NDcwMDAwMDAwMSwgNzYuNTA3NDE0NTE4MDE3MywgMjM2NzUuMF0sIFs5Ljg0OTc4NzIsIDc2Ljk3OTc5MTQsIDMwMTUuMF0sIFsxMS44NzYyMjU0LCA3NS4zNzM4MDQzLCA1NTQ5LjBdLCBbMTIuNDIxNzEzMDUwMDAwMDAxLCA3NS4xOTA0NDk3NzI2NjMyMSwgMjgwNy4wXSwgWzguOTAyMDMxNCwgNzYuNjM3NDEwMDE1Mzg0NjIsIDg0NjguMF0sIFs5LjYyODU3MDQ1LCA3Ni42NDU1MjUwMjkxNDc5LCA2NTU3LjBdLCBbMTEuMjQ0NjE0NCwgNzUuNzc1OTM3MiwgNjgyNS4wXSwgWzExLjEwNjg0NDc1LCA3Ni4xMDk5NTUxMDQ2NjY2MiwgNjMyNS4wXSwgWzEwLjc2OTE5ODksIDc2LjY1MTI0NjksIDY1MTYuMF0sIFs5LjI3LCA3Ni43OCwgMzg4MC4wXSwgWzguNTA1ODkwOSwgNzYuOTU3MDQ4MSwgMjI0NTUuMF0sIFsxMC41MjU2MjY0LCA3Ni4yMTMyNTQyLCAxMjU1Ny4wXSwgWzExLjcxNTIxOTAwMDAwMDAwMSwgNzYuMTI2OTAyOTQ2NTgxOTgsIDE0NTQuMF0sIFsyMy45MzQyOTYzLCA3Ni4xNDUyMzMxNDEyMjMxMiwgODI5MC4wXSwgWzIyLjI4NTkzOTU1MDAwMDAwMiwgNzQuMzU0Njg2MDg3NTIwOTIsIDc5NDMuMF0sIFsyMy4wMzUyNTk2NSwgODEuMzg4NDQzODQ3NDYzOCwgODY0MC4wXSwgWzI0LjU3NjE4OSwgNzcuNzMwMjg5NSwgODc0NC4wXSwgWzIxLjg2MTExODEsIDgwLjMyMTQxODcyODc0MjEyLCAyMzE0MS4wXSwgWzIxLjc1MTk0MzI1LCA3NC44OTE3NTE3NTMzODQ4OCwgMTQxODYuMF0sIFsyMS44Nzk2MTYyLCA3Ny44NzU2ODEyNDU2MzA3OSwgMTQ0NDIuMF0sIFsyNi41LCA3OC43NSwgMTY0NTIuMF0sIFsyMy4yNTMwOTIzLCA3Ny4zOTYyNzE4LCA0MzQ1My4wXSwgWzIxLjMxMTg4MzksIDc2LjIyOTE5OTIsIDE1MDkxLjBdLCBbMjQuNzUsIDc5Ljc1LCAxODYyOS4wXSwgWzIyLjEzOTgzMTA0OTk5OTk5OCwgNzguODA5NjQ0OTU0Njc5ODcsIDIwNTgzLjBdLCBbMjMuNzUsIDc5LjU4MzMzMywgMTU2MjUuMF0sIFsyNS43NSwgNzguNSwgOTAzOC4wXSwgWzIzLjAsIDc2LjE2NjY2NywgMjc4NTIuMF0sIFsyMi41LCA3NS4yNSwgMzY5MzguMF0sIFsyMi45MDczNjg2NTAwMDAwMDIsIDgwLjg4NTExNDI5Njg4MDI5LCA4MjAwLjBdLCBbMjEuNzUsIDc2LjU4MzMzMywgMjI1MDAuMF0sIFsyNC41LCA3Ny41LCAxNzQ3OS4wXSwgWzI2LjIwMzcyNDcsIDc4LjE1NzM2MjgsIDMyMDM5LjBdLCBbMjIuMzM4ODgyOCwgNzcuMDkyOTkzMywgOTQ5MC4wXSwgWzIyLjYwMDE1MDIsIDc3LjkyNjY0NTIxNDEzMTkyLCAxNTIwMy4wXSwgWzIyLjcyMDUxNjIsIDc1Ljg2ODExMjQsIDU1OTAyLjBdLCBbMjMuMTYwODkzOCwgNzkuOTQ5NzcwMiwgMzE1NjMuMF0sIFsyMi44ODU4NTIyNSwgNzQuNzI1MTMzNjg4MDQ1MTksIDE2OTcxLjBdLCBbMjMuODMzOTYyMSwgODAuMzkyNDU2LCAxNzU3OC4wXSwgWzIxLjgxODc3NDMsIDc1LjYwNjQ1NzcsIDIyMzcwLjBdLCBbMjIuNjg1MzY2MjUsIDgwLjU4MTUwNTYxNzExODg4LCA4NDU0LjBdLCBbMjQuMjY1MTMwNiwgNzUuMzg3MTgxOTM3MjUwMTIsIDExMzU2LjBdLCBbMjYuMTY2NjY3LCA3Ny41LCAxNzE0NC4wXSwgWzIyLjk0NjcwNDcsIDc5LjE5ODAyMjgsIDE0MDE4LjBdLCBbMjQuNjMwNDQ2NTUsIDc1LjE4MzM5NjE0NjM1NDU3LCAxNTg5MS4wXSwgWzI0LjUsIDgwLjI1LCAxMDQ3NC4wXSwgWzIzLjI1LCA3OC4wODMzMzMsIDI0Nzk0LjBdLCBbMjMuODcxNjcxMzUsIDc2Ljc3NDg5MDIwNDA4ODksIDEzNDg1LjBdLCBbMjMuNTAxOTU3NzUwMDAwMDAzLCA3NC45NTI4NDUxODY4MDA5NywgMTc4MzkuMF0sIFsyNC43NTkyNjY4NSwgODEuNjU1MDAwMjEwNzgzNDEsIDIxMDY1LjBdLCBbMjMuODA5NjEyMjUsIDc4Ljc1OTExMzYwNTc4NjM0LCAxOTU5Mi4wXSwgWzI0LjUsIDgxLjAsIDIwMzA3LjBdLCBbMjMuMTE1Njg4MiwgNzcuMDY2MjM5MzkxNzY1MDQsIDIxMDExLjBdLCBbMjIuMjc1ODc4OTAwMDAwMDAyLCA3OS43MjEwNDQ2NTEwNDcsIDE4NzYyLjBdLCBbMjMuNjUyMjc4NCwgODEuNDMwMjY0NDQwNTY5MiwgMTQ0ODIuMF0sIFsyMy4zNzA3NDEwNSwgNzYuNjIwNTE1Mjc0NzU3ODUsIDkzNzcuMF0sIFsyNS42NjU2NjExLCA3Ni42OTU0ODQzLCA4MTI5LjBdLCBbMjUuMzc1MjQxMjUsIDc3LjgyODExOTMyNjI5NzE0LCAyMDgwNy4wXSwgWzI0LjI1LCA4Mi4wLCAxNzI3My4wXSwgWzI0LjE5NzQ0MzIsIDgyLjY2NjE0NTMsIDEyNzI4LjBdLCBbMjQuODU0NTAyNzUsIDc5LjA0Njk4MTIzODgyODAxLCAxNTAwNy4wXSwgWzIzLjE3NDU5NywgNzUuNzg1MTQyMywgMjg0MzEuMF0sIFsyMy42NDMxOTMwNSwgODAuOTQyMzk1MjIwMDkyNTUsIDk0NjUuMF0sIFsyMy45MTY2NjcsIDc4LjAsIDE1Njc4LjBdLCBbMTkuMTYyNzcyNTAwMDAwMDAzLCA3NC44NTgwMjQzMDA4NTE5NSwgNDQ5NjcuMF0sIFsyMC43NjE4NjI0LCA3Ny4xOTIxNzE2MjUyNDYyMywgMTU2MDkuMF0sIFsyMS4xNTQ1NDExNSwgNzcuNjQ0Mjk2MTc5OTg3NDQsIDE1MzYxLjBdLCBbMTkuODc3MjYzLCA3NS4zMzkwMjQxLCA4NDcxNC4wXSwgWzE4Ljk5MDQwNTksIDc1Ljc1NDIyOTEsIDE3ODQ5LjBdLCBbMjEuMTIyODE1NDUsIDc5Ljc5MzI4MDI3MTQyNTg5LCAyNDM2MC4wXSwgWzIwLjU4MzMzMywgNzYuNDE2NjY3LCAxNzMyNi4wXSwgWzIwLjA5Njc1NTUsIDc5LjUwNDU0NzUyMjMwNjIxLCAyNzEwNy4wXSwgWzIxLjEzMDUwMTQ1LCA3NC40Nzg5MTE4MDU1MTIyOCwgODYzMC4wXSwgWzE5Ljc1OTA3MDM1MDAwMDAwMiwgODAuMTYyMjgwNzI1ODAxODIsIDE1NTI0LjBdLCBbMjEuNDU1MjI4LCA4MC4xOTYyNzI5LCAyOTUzMS4wXSwgWzE5LjU0MTQwOTY1LCA3Ny4xNzM3NjYwMTMxNzUxNSwgNTc4MS4wXSwgWzIwLjg0MzUxMTg1LCA3NS41MjU5MjY1ODc1NjAyNiwgMzU0NjQuMF0sIFsxOS45MTgyMzI4LCA3NS44Njg2MjQ2OTAwNDQzLCAyNTM1NC4wXSwgWzE2LjcwMjg0MTIsIDc0LjI0MDUzMjksIDYxMDIzLjBdLCBbMTguMzUxNDY4NTUsIDc2Ljc1NTEyMTIyMzA1MTMsIDEyMjA0LjBdLCBbMTguOTM4NzcxMSwgNzIuODM1MzM1NSwgMTQ3ODYxLjBdLCBbMTkuMTMwOTU3NjUsIDcyLjg4NTkzMDk1NDYwOTUyLCAyMDY1NDEuMF0sIFsyMS4xNDk4MTM0LCA3OS4wODIwNTU2LCA4NDQxOC4wXSwgWzIxOS4xNiwgNzcuMzIsIDEyNjA5LjBdLCBbMjEuMzY1OTk4OTQ5OTk5OTk4LCA3NC4yODQwMDM1ODE3ODc2NiwgMzY3My4wXSwgWzIwLjAxMTI0NzUsIDczLjc5MDIzNjQsIDYyMzExLjBdLCBbMTguMTY5ODQzOTUsIDc2LjExNzk2MzIxMTU5NTcyLCA5NDI4LjBdLCBbMTkuNjgwODYzODUsIDcyLjgyNTM3MzQyNTExMzQxLCA3NDkzNy4wXSwgWzE5LjI5MDMxMzY1LCA3Ni42MDI5MDM0MzQzMTIwMywgOTQ2MS4wXSwgWzE4LjUyMTQyOCwgNzMuODU0NDU0MSwgMjM0NDE1LjBdLCBbMTguNDkyODA5MiwgNzMuMTM4MDcwOTU0MjY1MzksIDU4MTI2LjBdLCBbMTcuMjgyNjA3OTAwMDAwMDAyLCA3My40NTY5Nzg3MDM5ODI2LCAxNTYzMi4wXSwgWzE2Ljg1MDI1MzQsIDc0LjU5NDg4ODUsIDMyOTExLjBdLCBbMTcuNjM2MTI4ODUsIDc0LjI5ODI3ODA3NjAxNzgyLCAyNjYwOS4wXSwgWzE2LjEzNTcxOTI5OTk5OTk5OCwgNzMuNjUyMjA4NjAxODM1ODQsIDc2MzEuMF0sIFsxNy44NDk5MDY2NSwgNzUuMjc2MzIwMjczNDg0NTcsIDM5Mjc4LjBdLCBbMTkuMTk0MzI5NCwgNzIuOTcwMTc3OSwgMjE5NjUwLjBdLCBbMjAuODI1NjIzMTUsIDc4LjYxMzE0NTQ5NTIyOTE5LCAxODI0Ny4wXSwgWzIwLjI4NzkyMDk1LCA3Ny4yMzYwNjI4MTA2MTAzNSwgOTAzNC4wXSwgWzIwLjE1LCA3OC4zNSwgMTA2NjMuMF0sIFsyNC41NjI0NjM2NDk5OTk5OTgsIDkzLjgwMTI0ODM3NDY1NDUsIDM0MjcuMF0sIFsyNC4zMTk3NDQyLCA5NC4wMjEwOTg4MjI1NDMyMSwgODE1LjBdLCBbMjQuMzc4NzA0LCA5My42OTcwMDE0OTUzMzk0NCwgMTYzNi4wXSwgWzI0Ljg1MTU0MDI1LCA5NC4wMDk0Nzk1NzcyMTU4MSwgMTAwMzEuMF0sIFsyNC43NTczMjYsIDkzLjg1ODQ3ODU4MDI1MTQ3LCAxMDcwMS4wXSwgWzI1LjM4MDQzOTYwMDAwMDAwMywgOTQuMDU2OTg1NDE3ODY1NSwgMTY2Mi4wXSwgWzI0LjkzNTQwMTMwMDAwMDAwMiwgOTMuNTY3OTExMTIyODk3MjQsIDI1OS4wXSwgWzI0LjYyNDU4NywgOTQuMDQyNDc5Mjg0MDI2ODQsIDQ1OTIuMF0sIFsyNS4xMDkyNjY4NDk5OTk5OTcsIDk0LjM4MjM5MjgxNjU0MTk3LCA4NzguMF0sIFsyNS42MTg0OTQ1NSwgOTAuNjM0MjE2MjE3Nzc3NDksIDczLjBdLCBbMjUuMjUxNTc2MDUsIDkyLjQ4NDA1MDA2OTQyMDE2LCAzNi4wXSwgWzI1LjQwNTY4MDg1MDAwMDAwMywgOTEuODM1NDY3NzEyOTg0NTUsIDg4MS4wXSwgWzI1Ljg2MTk4NDEsIDkwLjcwMTc3MDQ4ODM5MzQ4LCAxNDAuMF0sIFsyNS44NzY0MDI1NSwgOTEuODU2NzU3NDE2NDE2MDYsIDIwMy4wXSwgWzI1LjM0Nzk2OSwgOTAuNTY1Nzc3MjU5NjQ1NjMsIDE2LjBdLCBbMjUuNDQ4MDUyNjk5OTk5OTk4LCA4OS45OTg3ODUyOTY4NjE2LCA1Mi4wXSwgWzI1LjMzMDcyMjEwMDAwMDAwMywgOTEuMzI2NDUwMTM3ODYzMTksIDYzLjBdLCBbMjUuNjA3ODAxNzUsIDkwLjIwMzgwMTA2MTIyMjM1LCAxODIuMF0sIFsyNS40NDI1OTY1LCA5Mi4yOTg4Mjc0NzA5MTc3MywgMzMxLjBdLCBbMjUuNTgxNzM2OTUsIDkxLjI4OTEwMzA2MzcxMDE3LCAxODguMF0sIFsyMy43NDE0MDkyLCA5Mi43MjA5Mjk3LCAyMDU1LjBdLCBbMjMuNjkwODIzODUsIDkzLjM0ODM5MTE5OTEzNzc4LCAyNDcuMF0sIFsyNC4xODk0NzUyNSwgOTIuNzEzMzE4MTI4MTEyNCwgMTYyLjBdLCBbMjIuMzQ1OTY0OTUsIDkyLjgxMTk1MzYwNjg5NjksIDU4LjBdLCBbMjIuODk4NTUzLCA5Mi43NTE5MjI5MTcxOTg3NCwgMjMyLjBdLCBbMjMuNzU1OTA4NDUsIDkyLjQ1MTczNTY4ODMwNjM3LCA3My4wXSwgWzIyLjQ5OTU1MTgsIDkyLjk3Nzk4OTg4NzE3MDI3LCA1Ny4wXSwgWzIzLjM4NTg5MjMwMDAwMDAwMiwgOTIuOTMwNTk4OTA4MTc2LCAyNTEuMF0sIFsyNS45MTM1OTE0LCA5My43MjgzNzA3LCA3NzYuMF0sIFsyNS45LCA5NC43ODMzMzMsIDE0LjBdLCBbMjUuNzUsIDk0LjE2NjY2NywgMTYyLjBdLCBbMjYuNDkwMzE3MjUsIDk0Ljc3MjE0MjQ3MzAzMTM4LCAzLjBdLCBbMjYuNDc5NTg2MSwgOTQuNTEwNTI3NTQ3NjgyNzMsIDE0My4wXSwgWzI2Ljc1LCA5NC44MzMzMzMsIDIzLjBdLCBbMjUuNDc5MDc3MDUsIDkzLjcyNTYzNjU5MzEzMjA3LCA2LjBdLCBbMjUuNzUsIDk0LjUsIDUyLjBdLCBbMjYuMTY4NTA1NTAwMDAwMDAyLCA5NC44NTgxOTgyMTY1OTExNSwgMTM4LjBdLCBbMjYuMTY2NjY3LCA5NC4yNSwgODUuMF0sIFsyNi4wLCA5NC41LCAyLjBdLCBbMjAuODM4MjQyNiwgODUuMDk3Mzk0OSwgMjIyMy4wXSwgWzIwLjc1LCA4My4yNSwgMTkyOS4wXSwgWzIxLjUsIDg2Ljc1LCAyNTY0MC4wXSwgWzIxLjM0NzQ2MzcsIDgzLjY1NDI3OTIyMjgyMDQzLCAxNDU1LjBdLCBbMjEuMDYzMzI4OCwgODYuNTA1MzczMSwgNDQ0MC4wXSwgWzIwLjg0MTU2MTksIDg0LjMyMTMzMjEsIDIzNjkuMF0sIFsyMC40Njg2LCA4NS44NzkyLCAxMjU4Ni4wXSwgWzIxLjUzNzA3ODYsIDg0LjczMDExODQsIDE0Ny4wXSwgWzIwLjc1LCA4NS41LCAxMzQ1LjBdLCBbMTkuMTk1NTc0MTUsIDg0LjE5Mjk3NzM2MzQ2MzE3LCA2OTcuMF0sIFsxOS41LCA4NC41LCA5MDgyLjBdLCBbMjAuMjU5Mzg3MiwgODYuMTY2MDg3OCwgMTU5Ny4wXSwgWzIwLjgyNzc4OTE5OTk5OTk5OCwgODYuMzc5ODM1MjU5MTUxOTIsIDIyODQuMF0sIFsyMS44NzcwNTEyNSwgODQuMDA5MDUwNjM3MzUwMTksIDExODUuMF0sIFsxOS43NSwgODMuMCwgOTk0LjBdLCBbMjAuMTMwNjkzODUsIDg0LjA3OTA0NzYwNTkwMTU0LCA4NTkuMF0sIFsyMC41MDA0MDA4NSwgODYuNDMxNzA2MjI3OTY0NzgsIDEyNjguMF0sIFsyMS41ODkyMzIzLCA4NS42Njg3ODQxMzczNjkwNiwgMTIyNy4wXSwgWzIwLjIyNTY0NzE1LCA4NS41NjA1OTQ2NTEyNTIzMiwgMTU1NTkuMF0sIFsxOC43MjMyMDIzLCA4Mi42MDgxMTgyNzc4NzI2OSwgMTQ0OS4wXSwgWzE4LjM1ODM1NDM1LCA4MS44OTgyMzc0ODc0NzA0OCwgNTIzLjBdLCBbMjEuNzUsIDg2LjUsIDIwNjYuMF0sIFsxOS4yMjg4OTYsIDgyLjU2NjU3MTUsIDUyNy4wXSwgWzIwLjExNjA2NjYsIDg1LjExMTk1NjQzODQzNTczLCAxMzMxLjBdLCBbMjAuNzY5MDk5NzUwMDAwMDAyLCA4Mi41MTg2Mjg0NTg3NDEyMiwgMjc5LjBdLCBbMTkuODA3NjA4MywgODUuODI1MjUzOCwgNzUzNi4wXSwgWzE4Ljk2NTk5ODU1MDAwMDAwMiwgODQuMTkyODc0NTUwMTQ5OTIsIDE3MDcuMF0sIFsyMS40LCA4My44ODMzMzMsIDIwNjQuMF0sIFsyMC44NDUzODY0LCA4My45MDkwODMsIDkwOC4wXSwgWzIyLjI1LCA4NC41LCA0NjQwLjBdLCBbMzEuNjM0MzA4MywgNzQuODczNjc4OCwgMTA4MDQuMF0sIFszMC4zNzA0Njg1LCA3NS41MDQwMTc0MTg1NDgsIDM0NjQuMF0sIFszMC4xNzkxMTUzNSwgNzUuMDQ3MTAxNTYzMTI2MjgsIDk0MjkuMF0sIFszMC42MDA5MjUwNSwgNzQuNzk0Nzc0MjI4NDA0NDksIDQ0MjQuMF0sIFszMC42NjAwNzY0LCA3Ni4zODAwMjE2MzcxMDAyOCwgMzcxNy4wXSwgWzMwLjMzNjA5OTU5OTk5OTk5NywgNzQuMTE3OTQzMTkyMjI0ODEsIDI0OTIuMF0sIFszMC45NCwgNzQuNjIsIDQxNDQuMF0sIFszMS45MDQyMjk4OTk5OTk5OTcsIDc1LjIyNzM4MTQxMDk4OTM1LCAzNzc3LjBdLCBbMzEuNjA4NTc0MjUsIDc1Ljg0NjQ0MjQ2ODkwOTQ2LCA2ODc5LjBdLCBbMzEuMjkyMDEwNjUsIDc1LjU2ODA1NzcyMjUzOTExLCAyMzU4Ny4wXSwgWzMxLjM4NTI0MDk1LCA3NS4zMDU1MjI3Mzk4OTM5NSwgNzI2NS4wXSwgWzMwLjkwOTAxNTcsIDc1Ljg1MTYwMSwgNTU0NTguMF0sIFsyOS44NzY5OTcxLCA3NS40ODg5ODY2MTM3MDE4LCA0MTYzLjBdLCBbMzAuNzgzOTg2NiwgNzUuMTYwNTc0MjI1NDExNjIsIDczMjEuMF0sIFszMi4zMDE3MTA0MDAwMDAwMDUsIDc1LjY1ODY0MjQ2NjIyMDg0LCAxMDY4LjBdLCBbMzAuMjA5MDg3NCwgNzYuMzM5ODcyMDg1NjIyMSwgMTM0NzguMF0sIFszMS4wOTE2ODA4NSwgNzYuNTI3MjY3MzkxNjEzOCwgMjQ0Ni4wXSwgWzMwLjIwOTMzNjMsIDc1LjgxODQyMjQzNDA5MDksIDg3OTAuMF0sIFszMC42OTA0NDgxLCA3Ni43MTE2MzgyLCAxNjczMy4wXSwgWzMxLjEyNTI3OCwgNzYuMTE2Mzg5LCA0NTI4LjBdLCBbMzAuNDY5MiwgNzQuNTE4MiwgMzI0OS4wXSwgWzMxLjMyMTI0NTI1LCA3NC44NDEzMDYyOTM0MDc4NSwgMzEyNy4wXSwgWzI2LjQ2OTEsIDc0LjYzOSwgMjEyMDcuMF0sIFsyNy42MzkwNzcwNDk5OTk5OTcsIDc2LjYxNDQ1MjQ5MDIwNDUsIDIzMjcxLjBdLCBbMjMuNDkzMDc4OCwgNzQuMzQ4NDAyMzEzMTA2MjIsIDQzNTYuMF0sIFsyNC45MTcxNTEyLCA3Ni42OTY0MDMyMjQ4OTEzNiwgMzAwNS4wXSwgWzI1LjU4MTkwMzQsIDcxLjYxOTY2MjQyNzc3MTk2LCAxMDQzNS4wXSwgWzI3LjI2NTIxMjQ1LCA3Ny4zNjkxMjU1NDczOTEyMiwgODU3Mi4wXSwgWzI1LjQ4ODc3MzQ1LCA3NC42OTk2MTI4MzU4NDAyNCwgMjcyOTcuMF0sIFsyOC4wMTU5Mjg2LCA3My4zMTcxMzY3LCAyMDA1NC4wXSwgWzI1LjUsIDc1LjgzMzMzMywgMzk1NC4wXSwgWzI0LjcxODAyNjAwMDAwMDAwMiwgNzQuNDcyMTQ2OTcyMDA4NzYsIDg3NjIuMF0sIFsyOC4yMDYxNDQzLCA3NC42OTE5MDcyOTk2MzQ2MSwgODE0MC4wXSwgWzI2LjgwNDg2NTg1LCA3Ni40NDM3NDU2OTkyOTMyOSwgMTAyOTEuMF0sIFsyNi43MDA5ODk2LCA3Ny44OTY3ODU3LCAyNzU5LjBdLCBbMjMuNjY2NjY3LCA3My43NSwgMzcxMS4wXSwgWzI5LjUsIDc0LjAsIDE4MzU1LjBdLCBbMjkuMzY3MjAwMTUwMDAwMDAyLCA3NC4yOTgzNjM2NTA2NTkwMSwgMTY3ODQuMF0sIFsyNi45MTYxOTQsIDc1LjgyMDM0OSwgMTg1MTg3LjBdLCBbMjcuMDI4MDE2MTUsIDcwLjc3ODUwNTYyMzIwNzcsIDIzNDAuMF0sIFsyNS4zNDc2MDA2LCA3Mi42MjYwOTE2LCA1MjI0LjBdLCBbMjQuMzEzMjM2OCwgNzYuNTIyMjM2MjYxMjMwMDMsIDMxMTMuMF0sIFsyOC4xMjg3OTk1LCA3NS4zOTkyNTgxLCA5MDIzLjBdLCBbMjYuMjk2NzcxOSwgNzMuMDM1MTQzMywgNDA5NDAuMF0sIFsyNi41MTY2ODEwNSwgNzcuMDU3NzI5NzY1MTczNjMsIDQzNzYuMF0sIFsyNS4xOTY4MjU2LCA3Ni4wMDA4OTMzMDg4NTU1MiwgMjM1NTIuMF0sIFsyNy4wNjA3ODU5LCA3NC4xNzY2NzUzNzU4MjcxMiwgMTY0NjguMF0sIFsyNS42MDQwOTA4LCA3My40MTU2MDg3ODU1NjkwMiwgMTE3NjQuMF0sIFsyNC4wMzMzNDk1LCA3NC43ODAxMzE1LCAyNDU0LjBdLCBbMjUuMjkxMzE2MTUsIDczLjgyNDQ5MjQ3NDAzNjEsIDY1MjYuMF0sIFsyNi4yMjkxNDExNSwgNzYuMzA0NTMyNzcwMTc5MzIsIDY3MjUuMF0sIFsyNy42NjI4MjYwMDAwMDAwMDMsIDc1LjAyNzkyNjI4NjkxMzMxLCAxNTgwMS4wXSwgWzI0LjgxMTQwNDY5OTk5OTk5NywgNzIuODMwMDI1NzMxNjAyNDksIDU0NDguMF0sIFsyNi4xMjIxNDcyNSwgNzUuNjYzNzUzNzM5MzIyMzYsIDQ2NDcuMF0sIFsyNC41Nzg3MjEsIDczLjY4NjI1NzEsIDIzMDY1LjBdLCBbMjcuMzMzMzMzLCA4OC42NjY2NjcsIDUyOS4wXSwgWzI3LjY2NjY2NywgODguNSwgMjMuMF0sIFsyNy4zMzMzMzMsIDg4LjQxNjY2NywgMTI2LjBdLCBbMjcuMzMzMzMzLCA4OC4yNSwgMTc1LjBdLCBbMTEuMDc2MDM1OTUwMDAwMDAxLCA3OS4xMTc0NTUzODE4MjczOCwgMjc0NS4wXSwgWzEzLjA4MDE3MjEsIDgwLjI4MzgzMzEsIDE4MjUxMi4wXSwgWzExLjAwMTgxMTUsIDc2Ljk2Mjg0MjUsIDEwODAzMi4wXSwgWzExLjc0MjY5Mzc1LCA3OS43NTAzMDY0NDE3MTkzNSwgMTM0NjguMF0sIFsxMi4wOTY4MDQ3NSwgNzguMTkzMDQzMDEwMjY3MTYsIDEyMzkwLjBdLCBbMTAuMzMwMzI5OSwgNzguMDY3Mzk3OTA4NDY5NywgMTg4MzIuMF0sIFsxMS4zNjkyMDQ0MDAwMDAwMDEsIDc3LjY3NjYyNjg2ODQxNzkzLCA0NzEzOC4wXSwgWzEyLjk2NDcxNjMsIDc5Ljk4Mzk2ODYsIDcxMzMxLjBdLCBbOC4wODc5NjQxLCA3Ny41NDY3NDE0LCAyNDMyNC4wXSwgWzEwLjkzMDE1MjIsIDc4LjA4NDg1NDU0NTcyODg5LCAxMTYwNC4wXSwgWzEyLjUxODg4MzUsIDc4LjIyMDY1MzYsIDIzMDAyLjBdLCBbOS45MjYxMTUzLCA3OC4xMTQwOTgzLCA1NTAwMS4wXSwgWzEwLjgwNTYyNzYwMDAwMDAwMSwgNzkuODI0NjU5NzgzMDI0LCA2NDU2LjBdLCBbMTEuMjE5MTMxOSwgNzguMjM3Mzk4MDE3Nzk5NjMsIDI2MjAzLjBdLCBbMTEuMjI4NzcxNiwgNzguODE4MjU1NTQ5NjI3ODIsIDIzNDcuMF0sIFsxMC41LCA3OC44MzMzMzMsIDg4MjkuMF0sIFs5LjM4OTU1MjMsIDc4Ljg1OTA3MDcxNTIxNDk4LCA1NTg2LjBdLCBbMTEuNjYxMjAxMiwgNzguMTYwMjQ5OCwgNjM2MTYuMF0sIFs5Ljg1MTIzMSwgNzguNTMwNDcxNTQ4MjA3MTYsIDc2MjguMF0sIFsxMC43ODYwMjY3LCA3OS4xMzgxNDk3LCAxMzc5MS4wXSwgWzExLjQ5MTYsIDc2LjczMzcsIDI1NjUuMF0sIFs5Ljk2OTY2NDMwMDAwMDAwMiwgNzcuNDc0MjAwNDg1MjQ4MjIsIDk1MDQuMF0sIFsxMy4xMzAxNDc1NSwgNzkuOTI0MzUzODYyNTQ5NjcsIDU1MzU2LjBdLCBbMTAuNzM2MTg2MDUsIDc5LjYzMzE4NjU5NDM3NjI3LCA1Mzk4LjBdLCBbMTAuODA0OTczLCA3OC42ODcwMjk2LCAyNDI3OC4wXSwgWzguODA4MjM0MiwgNzcuODExNDg0MywgMjU0ODAuMF0sIFsxMC43ODMyMjcwNSwgNzcuNTI2MDQ3ODAwOTY4NDQsIDg5NDk0LjBdLCBbMTIuMjI3MjEyOTUsIDc5LjA3MDE1NTU0OTA2MTY3LCAxMTc4NC4wXSwgWzguNzIzNTU3NywgNzguMDI1MTUzOTg5MzAzODcsIDE3MzU3LjBdLCBbMTIuNzk0ODEwOSwgNzkuMDAwNjQxMDk2ODU0OSwgMjgyNDcuMF0sIFsxMS45Mzk4Mjg1LCA3OS40OTQ1NjQ1LCAxNDA4Mi4wXSwgWzkuNTIwODkzNiwgNzcuODc4NDU2Mzk2MTg2NSwgMjY3NzguMF0sIFsxOS41LCA3OC41LCA0MTI4LjBdLCBbMTcuNzE1MzQ1MjUsIDgwLjU3MTQ5NzYxNzc4NzEyLCAxOTI1LjBdLCBbMTcuMzg4Nzg1OTUsIDc4LjQ2MTA2NDczNDUzMTQ2LCA1ODA1MC4wXSwgWzE4Ljc5NzEyNjksIDc4LjkyMjQ3MjcsIDI5NTEuMF0sIFsxNy43NTU3OTE5LCA3OS4xMzIzODk2MTE3OTI0NSwgODcwLjBdLCBbMTguNTE1OTg3MTk5OTk5OTk4LCA3OS45NjkzOTY1OTc5MjUyOCwgMTY5Ni4wXSwgWzE2LjA5OTkyMDIsIDc3LjczNDE1ODM1MDc3NTIzLCAxMjc2LjBdLCBbMTguMzE2NTUxLCA3OC4wNTM5MzgwODA0MzM0OCwgMzA0NC4wXSwgWzE4LjQzNDY0MzgsIDc5LjEzMjI2NDgsIDIwNzc4LjBdLCBbMTcuNSwgODAuMzMzMzMzLCA3Mjk1LjBdLCBbMTkuMzYxOSwgNzkuMjkzLCA0NjIuMF0sIFsxNy43MTM4OTgzLCA4MC4wNDEzNDI1Mjc2NzIyNywgMTI4Ny4wXSwgWzE2LjY5NjU2ODU1MDAwMDAwMiwgNzcuOTU5MTE0NjIwODk2OTMsIDEyMzQwLjBdLCBbMTguODc2MTc5NSwgNzkuNDQ0OTY5NiwgMjEyOC4wXSwgWzE3LjkzNzUwOTUsIDc4LjIxMTc0NDk4NTEwNTg0LCA1ODIyLjBdLCBbMTcuNTM0OTgzNDUsIDc4LjUyNDYzODczNTAyMzgxLCAyODM4NS4wXSwgWzE2LjQxNTc2MjY0OTk5OTk5OCwgNzguNjgzMDQzMzMxMzI4MjksIDE3MzguMF0sIFsxNi44NTc5NjM2LCA3OS4yMTc0OTM1MzMxNTU0OCwgMTI3NTguMF0sIFsxOS4wOTE1MjA5NTAwMDAwMDMsIDc4LjM5NjYwODk4NDI5MDY1LCAxNDg3LjBdLCBbMTguNzUsIDc4LjI1LCAxNzQxMi4wXSwgWzE4LjYyMDY1MzA1LCA3OS40OTUwMTcxMDg4Mjg5NiwgMzY4MC4wXSwgWzE4LjQ1MjExNiwgNzguNzY0NTU4MzcxNTg3MzcsIDM3NTkuMF0sIFsxNy4zODkxLCA3Ny44MzY3LCA0ODkyMi4wXSwgWzE3Ljg2ODU5MjA1LCA3Ny44MjI4MTk5NTA3MzgxLCA0MTUwLjBdLCBbMTguMDA1NTg1MDk5OTk5OTk4LCA3OC44OTYxMTMwNDk2OTk2MSwgMjc2Mi4wXSwgWzE3LjA4MDAxMzg1LCA3OS43OTI1MzI5NzQyNDI3LCA0MTI0LjBdLCBbMTcuMjcwMjg1NSwgNzcuNzQ1Mjk3MDA2MzczODIsIDEzNTMuMF0sIFsxNi4yODUyOTQzOTk5OTk5OTgsIDc3Ljk4NjQ0NzI3MDIxMjg2LCAxMjIyLjBdLCBbMTcuOTgwNjA5NCwgNzkuNTk4MjExNSwgMTI0MzkuMF0sIFsxNy45NDg1NDI1NSwgNzkuODE2MTIzOTkzNTk3NDIsIDMxMTMuMF0sIFsxOC4wMjYyNTY5NSwgNzkuNDY0NDQ0OTM5OTY2MzQsIDEzNzE5LjBdLCBbMTcuNDI4MTk2NjUsIDc5LjA5MDQ5MDk3NjEyMTA1LCAyMjU5LjBdLCBbMjMuODI1MzE1OCwgOTEuOTczMjIyNjcxNjMzMywgMzc2LjBdLCBbMjMuNDYxMzM2OSwgOTEuNjA3MTgwMDM4MTc3MjMsIDQ1Ni4wXSwgWzI0LjA0NjczODM1LCA5MS42MjMyMDk1MzY0Njc5OCwgNDcwLjBdLCBbMjQuMDkyMTQ3NTk5OTk5OTk3LCA5Mi4yNDUxMjEzNTAxOTg0NywgNzA4LjBdLCBbMjMuNTMwNzQ0NTUsIDkxLjMwMTU2NTM5MDMwNTczLCAzMzMuMF0sIFsyMy41LCA5MS42NjY2NjcsIDUwMi4wXSwgWzI0LjMxNjc0NTgsIDkyLjA2NjUwMjI0MzM5MTg1LCAzMDIuMF0sIFsyMy45MTY2NjcsIDkxLjUsIDIzOTAuMF0sIFsyNy4xNzUyNTU0LCA3OC4wMDk4MTYxLCA2NjQ0Ni4wXSwgWzI3Ljg3Njk4OTc1LCA3OC4xMzcyOTAyNzYwMDk5NCwgMTQ0MTkuMF0sIFsyNi40Njg0LCA4Mi42OTE1LCA1ODc0LjBdLCBbMjYuMzQ3MzgzMTUsIDgxLjYyMzg3ODMyMDc4ODQsIDIyNDcuMF0sIFsyOC45MjMzOTY5LCA3OC40ODgzMTY5MzI2MjY4NCwgMzYwMi4wXSwgWzI2LjY1NTczMzk1LCA3OS41MTUwNDcwMTE4MTgzMSwgNTM5Ni4wXSwgWzI2LjAyMjY5Njc1LCA4My4wMjg4NzM0MzExNDg0OCwgNzM4OC4wXSwgWzI4Ljk3MjQwNDA1LCA3Ny4zMzMxNTI4NDkzODIyNCwgMjkxNi4wXSwgWzI3LjczMzY5NTgsIDgxLjQ3NzMyMTI3NjYxMDU4LCAyMDQ5NC4wXSwgWzI1Ljg3NzkzMjU0OTk5OTk5NywgODQuMTE5OTU5MzE0NjAzNzksIDMxNjcuMF0sIFsyNy40NTA0ODA1LCA4Mi4zOTU0MTc3NDA3MzEzMiwgMzUzMy4wXSwgWzI3LjEzMDMzNDQsIDgwLjg1OTY2NiwgMjc1NS4wXSwgWzI2LjkzODIzMTA1LCA4MS4zODYwOTc2MTIwNDk5MSwgMTc5OTQuMF0sIFsyOC40NTc4NzYsIDc5LjQwNTU3MDkzNzQzMDU4LCAxNzAyNC4wXSwgWzI3LjI1LCA4My4wLCA1MDQ5LjBdLCBbMjUuNDIyOTIyNzQ5OTk5OTk4LCA4Mi40ODg0MzcwODA4ODU3NSwgMTUwNjcuMF0sIFsyOS40MDYwNDg5NSwgNzguNDgwODc4Mzk5MTc4NzgsIDQwNTUuMF0sIFsyOC4wNjgzMTE2NSwgNzkuMDQ2MDczMTIwNzkyOTMsIDMxMDEuMF0sIFsyOC41MjQ2NzEyLCA3Ny41OTk2NTI5LCA4ODgwLjBdLCBbMjUuMTI2NTc3NywgODMuMjQ5NjU1NzczMzM5MSwgNjM2Ny4wXSwgWzI1LjIxOTUyMDcsIDgxLjEwMjI4MjQyNSwgNDE3NC4wXSwgWzI2LjQyMzg0NzE1LCA4My43NjI3MzE2OTAyODcyNywgNTI5OS4wXSwgWzI3LjU1NDExNDYsIDc4LjYwMDc4NjQwMzA0NjM0LCAyNjgwLjBdLCBbMjYuNzE4MzI0MzUwMDAwMDAzLCA3OS4wOTAyNTM3NTAyNTE2MywgODY3OS4wXSwgWzI2LjYzODA3NTU1LCA4Mi4wNTkwMjQzNDM3ODYyNSwgMzM2MC4wXSwgWzI3LjQzNzE5Mzg1LCA3OS40ODkxMjk0NjUwMzEyMywgMzg4My4wXSwgWzI1Ljg0MzUzOTUsIDgwLjkxODAwMzk3MDg4NTAxLCA1MjczLjBdLCBbMjcuMTc3MzY2MzUsIDc4LjM4OTkxMTk3OTUxMTgyLCA5MjM4LjBdLCBbMjguMzY3NjA5NzUsIDc3LjU5NzQwMzI2MzY5OTAyLCAzMjE5OC4wXSwgWzI4LjcxMTI0MSwgNzcuNDQ0NTM3MiwgMzExNTQuMF0sIFsyNS42MDM1MDg0MDAwMDAwMDMsIDgzLjUwNzQ1NDA0ODg3MTM4LCAxODE3MC4wXSwgWzI3LjEwOTY2NjksIDgxLjkxODMyOTEyMTg4MTMsIDEzMDI0LjBdLCBbMjcuMDIzMjA0OSwgODMuMjY3Njc2OCwgMTA4NjYuMF0sIFsyNS43NSwgODAuMCwgMjM3NS4wXSwgWzI4Ljc0MDYxMjQ5OTk5OTk5NywgNzcuODM1NDI1NTY1MTkwMDksIDQyMTAuMF0sIFsyNy4zMzg1NzY2NSwgODAuMDk3NTI2NDY0MTcxOSwgMzQzNi4wXSwgWzI3LjU3MzI0MzI1LCA3OC4xMTE3Mzg2Njc1MTcxNiwgMTQ4MDQuMF0sIFsyNi4xMDU1NDYsIDc5LjQyNzY1MjY0NzgyMDc2LCAyNDg4LjBdLCBbMjUuNzk1NTkyNywgODIuNDg4MzQwOTc1MDQzODUsIDQxODc1LjBdLCBbMjUuNTMxMDMxMTQ5OTk5OTk3LCA3OC42NTI2ODkwMTYwNTM0OSwgODQ3OS4wXSwgWzI2Ljk5OTY5NzQwMDAwMDAwMiwgNzkuNjg4NjEyMTI3NzU4NzQsIDQ1MTMuMF0sIFsyNi40NjAwNDY1LCA3OS44NTU1MDk5OTYxNTU5MiwgNDM5MS4wXSwgWzI2LjQ0MDIxMTksIDgwLjI2OTMwNTEzNDE2NjM3LCAzMjYyNi4wXSwgWzI3Ljg4Mzg0NjA1MDAwMDAwMiwgNzguNjM0ODkwMDM3NDc4NzMsIDE5NTkuMF0sIFsyNS41MzYwOTQ1NSwgODEuNDQ2NzI4Mzg3NjQ2MDUsIDEyOTMuMF0sIFsyNi45MjM4MzQ4NSwgODMuODMyNTUwMjY5MjU3OTgsIDM3OTIuMF0sIFsyNy45ODUwNjAxNTAwMDAwMDIsIDgwLjc1Mzg0NTM4MzU3NjQ5LCA1ODUzLjBdLCBbMjQuNzAwMzg0ODUsIDc4LjUxODY2NzU4NjQ4MzQ5LCAzMzY1LjBdLCBbMjYuODM4MSwgODAuOTM0NjAwMSwgMzY3MTEuMF0sIFsyNy4wOTI1NDIyNSwgODMuNTY2OTYyNjEzMzgyNTcsIDExNDcyLjBdLCBbMjUuMzYxNzY4ODUsIDc5LjcwNDk5NDc1MzMxODc3LCA5MTkuMF0sIFsyNy4yMDk4MjE5LCA3OS4wNDgxMzcwMDg0Nzk0NCwgNzI2Ny4wXSwgWzI3LjYzMzMzMywgNzcuNTgzMzMzLCAzMjY5My4wXSwgWzI2LjAzNzY4NzgsIDgzLjQ5NzkzNDE4NDk5MDQ3LCA0OTg1LjBdLCBbMjguOTE2NjY3LCA3Ny42ODMzMzMsIDMxNzM2LjBdLCBbMjQuOTM1NjM0NywgODIuNjQ3NzAxMjk4MTE2NDksIDI2MDY3LjBdLCBbMjguODYzODQyNCwgNzguODA1Nzc4MzMwOTExMDQsIDE5ODY4LjBdLCBbMjkuNDQ4MDA2MzUsIDc3Ljc0MDY4NTAyNTY4NjcyLCA3NDAxLjBdLCBbMjguNDk1MjA3NjQ5OTk5OTk4LCA4MC4xMDc1NDA4MDAxODA3NywgMjcwOC4wXSwgWzI1Ljc1LCA4MS43NSwgODgyMC4wXSwgWzI1LjQzODEzMDIsIDgxLjgzMzgwMDUsIDUwMDA2LjBdLCBbMjYuMjUsIDgxLjI1LCAzODQzLjBdLCBbMjguNzk0MDY4MjUsIDc5LjE4NTkzMDQ0NTg1NTQsIDU5NjQuMF0sIFsyOS45ODgwNzc0LCA3Ny41MDgxMjk5NDcwNTEyLCAxMTE4OS4wXSwgWzI4LjYxODc1MjU1LCA3OC41NTA4NzQwNTQwNDgwNCwgMjIzMi4wXSwgWzI2Ljc2NzIsIDgzLjAzNjEsIDU5MTcuMF0sIFsyNy45MTI2MzMxNDk5OTk5OTgsIDc5Ljc0NjU2Mjk0ODY5ODI2LCAyNjkzLjBdLCBbMjkuNTAwODgxNiwgNzcuMzQ4MzgyNTg1NDA4NjEsIDM2OTQuMF0sIFsyNy41MDgzNDI5LCA4Mi4wMjYzNTY5LCAxMTcxLjBdLCBbMjcuMjk5MSwgODMuMDkyOCwgOTc5My4wXSwgWzI3LjUwNDYzOTIsIDgwLjgyOTQ2NTgzNDIwODE1LCA3OTg5LjBdLCBbMjQuMzg2NTc1MjUsIDgzLjA3ODYzMDYxMjc4NTA1LCAyNDA1Ni4wXSwgWzI2LjI0MjUxMDg1MDAwMDAwMiwgODIuMjk2MTY5MzE2ODU5MTgsIDUwMzkuMF0sIFsyNi41NzU1MDM2NSwgODAuNjEzNzYxNzc3ODI4NTYsIDQ4NjEuMF0sIFsyNS4zMzU2NDkxLCA4My4wMDc2MjkyLCAyNjI1Mi4wXSwgWzI5LjcwMzA5NDk5OTk5OTk5OCwgNzkuNDMzMTcwMjMzMjY3MTYsIDEwNzAuMF0sIFszMC4wMDg2NzIyLCA3OS45MzAyOTY3OTUwNDQ3LCA3NzAuMF0sIFszMC40OTk2MzIzMDAwMDAwMDIsIDc5LjYxODc5MjQ1OTQ0NDA0LCA2ODYuMF0sIFsyOS4yMzYzMTMxNSwgODAuMTAxNzIwNzY1MTIwNzUsIDczOS4wXSwgWzMwLjMyNTU2NDYsIDc4LjA0MzY4MTMsIDkzMTMuMF0sIFsyOS45Mzg0NDczLCA3OC4xNDUyOTg1LCA2ODE1LjBdLCBbMjkuMjAzMTM5MiwgNzkuNDE3Njk0NSwgMjk5OC4wXSwgWzI5Ljg0NTkxMTE1LCA3OC43MDc2Njc0NjMyMDU0NywgNDc0NC4wXSwgWzI5LjU1NzAxNDA1LCA4MC4yMzE4MzUyMDcyOTIxLCA5NjkuMF0sIFszMC4yODQ0LCA3OC45ODExLCA2MjMuMF0sIFszMC41LCA3OC42NjY2NjcsIDEyMDEuMF0sIFsyOS4wNDgwOTg3OTk5OTk5OTgsIDc5LjQzMTMxMzMyMjMzNTcsIDc4MzYuMF0sIFszMC45NjU0MjE0LCA3OC42MzM2ODczMTE4Nzk2MywgOTQ3LjBdLCBbMjYuNDg1MTU3MywgODkuNTI0NjkyNiwgNzIwMy4wXSwgWzIzLjEzMTk1NDI1LCA4Ny4yMDczOTcyMDM2NzUwNywgNDU1MS4wXSwgWzI0LjAsIDg3LjU4MzMzMywgMTUxMTYuMF0sIFsyNi4zNDUyLCA4OS40NDgyLCA2NDkzLjBdLCBbMjUuMzg3MDMwODUsIDg4LjUwNDcxNTA0OTk5OTk5LCA1NzA4LjBdLCBbMjYuODM0OTAwMiwgODguMzA3MTQ5NTgyNTI3MDcsIDMyMzguMF0sIFsyMi40MjA3MDI1LCA4Ny4zMjY5OTYzLCA1MzU2LjBdLCBbMjIuOTA1MjExNCwgODguMzc2MDYzOSwgODEwMC4wXSwgWzIyLjU4ODIyMTYsIDg4LjMyMzEzOSwgMTM1MDIuMF0sIFsyNi42MjY0ODM2NDk5OTk5OTcsIDg4LjczNDA3NzAxNDY4OTkzLCAzMTYzLjBdLCBbMjIuMzc3MDYzNiwgODcuMDQ4NjcxNzczNjIyMDMsIDEzOC4wXSwgWzI3LjA3MTY5LCA4OC40NzI5LCAxMzYuMF0sIFsyMi41NzI2LCA4OC4zNjM5LCAyODY5NC4wXSwgWzI1LjAwNTc0NDksIDg4LjEzOTg0ODMsIDYxMTkuMF0sIFsyNC4xNzQ1OTkzLCA4OC4yNzIxMzM1LCA3MTY2LjBdLCBbMjMuNDg0NTQxMjUsIDg4LjU1Njc2MzA3NDcwNTM2LCA1Nzk3LjBdLCBbMjIuNjE2OCwgODguNDAyOSwgMjM5MTYuMF0sIFsyMy42NDI2MDc3NSwgODcuMTY0NDgxMjYwNjgzMjMsIDIzMzkuMF0sIFsyMy4zOTE3MTcsIDg3LjkwNjIxMjEyMzU1NDYyLCAyNzg3OS4wXSwgWzIzLjM3MTE2NDYsIDg2LjQxODkzNTY0Mzc0MDc4LCAyNjA3LjBdLCBbMjIuMTM1MiwgODguNDAxNiwgMjQxOTYuMF0sIFsyNS44NzA3OTU4LCA4Ny45NjI1OTcyODQ0OTM5MSwgMTk4My4wXSwgWzIyLjQyMDcwMjUsIDg3LjMyNjk5NjMsIDU5OTcuMF0sIFs3LjEyMDUsIDkzLjc4NDIsIDEzMi4wXSwgWzEyLjYxMTIzODY1LCA5Mi44MzE2NTQwNjQxNDkyNiwgNjgxLjBdLCBbMTEuNzQwMSwgOTIuNjU4NiwgNTIxNy4wXSwgWzMwLjcxOTQwMjIsIDc2Ljc2NDY1NTIsIDEwNzI4LjBdLCBbMjAuMTgwOSwgNzMuMDE2OSwgNDY2OS4wXSwgWzIwLjQyMDAwNDg1LCA3Mi44NjM3NjI5MDMwMDU2NiwgMjYxMy4wXSwgWzIwLjcxNzk4NTc0OTk5OTk5NywgNzAuOTMyMzk5MjQwOTIyLCA4OS4wXSwgWzEuMzA5MjE3MywgMTAzLjg1MzM0MzEsIDk2NzMuMF0sIFsyOC42NTE3MTc4LCA3Ny4yMjE5Mzg4LCAxOTAyMy4wXSwgWzI4LjYxNDE3OTMsIDc3LjIwMjI2NjIsIDE1ODU0LjBdLCBbNDIuMjc4MTQwMSwgLTc0LjkxNTk5NDYsIDExMzE2LjBdLCBbMjguNjUxNzE3OCwgNzcuMjIxOTM4OCwgNzYwMy4wXSwgWzQyLjI3ODE0MDEsIC03NC45MTU5OTQ2LCAyNDc2NS4wXSwgWzI4LjY3MzMzMzMsIDc3LjI4OTAyNDgsIDIzNjkuMF0sIFszNC44OTM1NTA1LCAxMjguNjg5MDAzOSwgMjIwMDcuMF0sIFsyOC42NTE3MTc4LCA3Ny4yMjE5Mzg4LCAzMjIzLjBdLCBbMjguNjUxNzE3OCwgNzcuMjIxOTM4OCwgMTU0NTguMF0sIFsyOC42NTE3MTc4LCA3Ny4yMjE5Mzg4LCAyMTAxMC4wXSwgWzMzLjc0NjEwOTk1MDAwMDAwNSwgNzUuMTg1NDQ3NTMxNDMyNTQsIDI3Ny4wXSwgWzMzLjkwOTkzMTkwMDAwMDAwNCwgNzQuNjI5MTQ0MjU3Nzk4MSwgODEwLjBdLCBbMzQuNDU2MzIzNCwgNzUuMTgzNDQ1NzgyNDI5NDYsIDg2LjBdLCBbMzQuMjA1MDA1NjUwMDAwMDA0LCA3NC4zNjIyMTA4MTExMTU5NywgNjczLjBdLCBbMzMuMTc3NDMwNywgNzUuNTY3NTIyODE4OTM3ODksIDYwLjBdLCBbMzQuMTkzNDI1NTAwMDAwMDA0LCA3NC44MzQzNDk3NjgyNjY3OCwgMTgxLjBdLCBbMzIuNzE4NTYxNCwgNzQuODU4MDkxNywgMjU1MC4wXSwgWzMzLjk3NTkyNDEsIDc2LjUwMjM1MDM5NTU3MTYxLCA0Ni4wXSwgWzMyLjU4MzMzMywgNzUuNSwgNDU5LjBdLCBbMzMuMjc2MTE3MSwgNzUuODE2OTIzODc4MTQwMywgMTExLjBdLCBbMzMuNjY5ODAwNTUwMDAwMDA1LCA3NS4wMTQ4MDIzMjIxNzM4OSwgNzUuMF0sIFszNC42MTEwMjM1OTk5OTk5OTYsIDc0LjIzODQ3ODY2NzMxOSwgNjMuMF0sIFszNC4xNjM0MjcxLCA3Ny41ODQzNDM3LCA5My4wXSwgWzMzLjc2NzAwMDQsIDc0LjA5NTcxNDMsIDgyMC4wXSwgWzMzLjg5ODI5MzU1LCA3NC44OTYzNDA5Mzc5MjAxNywgMTc5LjBdLCBbMzMuMjUsIDc0LjI1LCAzMDguMF0sIFszMy4yMTE2MzcxMDAwMDAwMDQsIDc1LjIwNTA5ODkwNzUyODE4LCAzNC4wXSwgWzMzLjA2OTEwMzE1MDAwMDAwNCwgNzQuODM3ODY5ODIwODg4OSwgODkuMF0sIFszMi41NzU5MTE3NSwgNzUuMTIwMzQ0NTU1MzAwNjMsIDQwMS4wXSwgWzMzLjcxODgwNywgNzQuODMzMzMxMSwgNDEuMF0sIFszNC4wNzQ3NDQ0LCA3NC44MjA0NDQzLCAxMjY0LjBdLCBbMzMuMCwgNzUuMTY2NjY3LCAxODQuMF0sIFszNC40MjA4OTQzNSwgNzUuNjY1NDg1OTgxMjE4NjEsIDQwLjBdLCBbMzQuMTY0MjAyOSwgNzcuNTg0ODEzMywgMzcuMF0sIFsxMC41NjY3LCA3Mi42NDE3LCA5OS4wXSwgWzEwLjkxNTcxLCA3OS44Mzc1NzYxLCA5MjEuMF0sIFsxMS43MDIxOTc4LCA3NS41MzY0NzAxLCA0OC4wXSwgWzExLjkzNDA1NjgsIDc5LjgzMDY0NDcsIDkzMDUuMF0sIFsxNi43MzMzNjY2LCA4Mi4yMTQ1MTY0LCA1Ny4wXV0sCiAgICAgICAgICAgICAgICB7ImJsdXIiOiAxNSwgIm1heCI6IDMxMjAyNC4wLCAibWF4Wm9vbSI6IDEsICJtaW5PcGFjaXR5IjogMC4yLCAicmFkaXVzIjogMTd9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiNTA3ZjM1MGNjZjQ4NTc5Njk3NTVmMjg0OTgwNDBmKTsKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python

```

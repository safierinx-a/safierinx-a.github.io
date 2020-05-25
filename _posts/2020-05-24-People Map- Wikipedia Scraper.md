---
title: "People Map of India: Part 1 (Scraper)"
---
I was going through [The Pudding](https://pudding.cool/) yesterday and came across their ["People Map of the UK"](https://pudding.cool/2019/06/people-map-uk/) and ["People Map of the US](https://pudding.cool/2019/05/people-map/) posts. These seemed pretty cool and since I've been trying to pick up making visualizations using JavaScript, I thought this was the perfect thing to attempt to replicate. So I decided to do the same for India.  
  
Before getting to the visualization, there is data collection so I decided to follow their methodology of using the "People by City" listings on Wikipedia. However. these lists aren't exhaustive and there are some more links but for the sake of brevity I decided not to try and scrape those. Those were a bit too hard to scrape, I've left the link to the collection of those pages, let me know if you see a good way to scrape the names from those pages!

###  Importing Dependencies


```python
import requests
import pandas as pd
import bs4
import wikipedia
from time import sleep
import pageviewapi.period
from tqdm import tqdm
```

### Retry function
Let's define a function that puts the script to sleep for exponentially incrementing timesteps so as to make sure we don't get a "Too Many Requests" error.


```python
def retrieve(url):
    page = ''
    l = 0
    while page == '':
        try:
            page = requests.get(url)
            break
        except:
        #    print("Connection refused by the server..Let me sleep for", pow(2,l), "seconds")
            sleep(pow(2,l))
         #   print("Was a nice sleep, now let me continue...")
            continue
    return page
```

### Define Number of Views function
I used the pageview API in python to collect the total views over the past five years. Again, I put in an exponentially incrementing sleep function in case we get an error for making too many calls.


```python
def viewj(person):
    r = ''
    l = 0

    while r == '':
        try:
            r = pageviewapi.period.sum_last('en.wikipedia', person, last = 365*5,
                        access='all-access', agent='all-agents')
            break
        except:
            print("Let me sleep for", pow(2,l), "seconds")
            print("ZZzzzz...")
            sleep(pow(2,l))
            l+=1
            if l > 10:
                r = 0
                print(person, "views error")
                break
      #      print("Was a nice sleep, now let me continue...")
            continue
        
    return r
            
```

### Scraping From Wikipedia
Using BeautifulSoup and the requests module, I scraped the people listed under each city in the link and discarded all of them except the person with the highest number of views in each city.


```python
central_url = "https://en.wikipedia.org/wiki/Category:People_by_city_in_India"
s = requests.get(central_url)
data = pd.DataFrame(columns = ['City', "Person", 'Number of Views', 'Description', 'link'])
soup = bs4.BeautifulSoup(s.content, 'lxml')
raw = soup.find_all('div', {'class':"mw-category-group"})
c = [] #Array of cities
p = []  #Array of most viewed people corresponding to those cities
v = []  #Array of thee views those people got
d = [] #Descriptions of the people
miss = [] 
err = []
li = [] #Links to their profiles
```


```python
for i in tqdm(range(2,len(raw))): #First two links are redirects to other categorizaations
                    #We basicallu just want to deal with the letters
        l = raw[i].find_all('a') #Get links within the letter categories
        for j in l:
            categ_link = "https://en.wikipedia.org"+j['href']       #Get link
            city = j['title'][21:] #Get rid of the "Category: People from " bit
            t = retrieve(categ_link)
            tsoup = bs4.BeautifulSoup(t.content, 'lxml')
            try:
                r = tsoup.find_all('div', {'class':"mw-category-group"})
                select = r[0].find('a')['title']
                max_v = viewj(select)

                for k in r:
                    person = k.find('a')['title']
                    if viewj(person) > max_v:
                        select = person
                        max_v = viewj(person)
                        people_link = "https://en.wikipedia.org"+k.find('a')['href']
                        try:
                            description = wikipedia.summary(select)
                        except:
                            description = "Error"
                
                miss.append(select)
                views = viewj(select)
                c.append(city)
                v.append(max_v)
                p.append(select)
                d.append(description)
                li.append(people_link)
                #print(city, person, views)
            except:
                r = tsoup.find_all('div', {'class':"mw-content-ltr"})
                r = r[len(r)-1].find_all('a')
                select = r[0]['title']
                max_v = 0

                for k in r:
                    person = k['title']
                    if viewj(person) > max_v:
                        select = person
                        max_v = viewj(person)
                        people_link = "https://en.wikipedia.org"+k['href']
                        views = viewj(select)
                        try:
                            description = wikipedia.summary(select)
                        except:
                            description = "Error"
                miss.append(select)
                views = viewj(select)
                c.append(city)
                v.append(max_v)
                p.append(select)
                d.append(description)
                li.append(people_link)

               # print(city, person, views)
sleep(120)              
                
                
```


```python
data['City'] = c
data['Person'] = p
data['Number of Views'] = v
data['Description'] = d
data['link'] = li
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
      <th>City</th>
      <th>Person</th>
      <th>Number of Views</th>
      <th>Description</th>
      <th>link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Agartala</td>
      <td>Dipa Karmakar</td>
      <td>1345973</td>
      <td>Dipa Karmakar (born 9 August 1993) is an India...</td>
      <td>https://en.wikipedia.org/wiki/Dipa_Karmakar</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Agra</td>
      <td>Mumtaz Mahal</td>
      <td>2758109</td>
      <td>Mumtaz Mahal (Persian: ممتاز محل [mumˈt̪aːz mɛ...</td>
      <td>https://en.wikipedia.org/wiki/Mumtaz_Mahal</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ahmedabad</td>
      <td>Abdul Latif (criminal)</td>
      <td>2087052</td>
      <td>Abdul Latif was an underworld figure in Gujara...</td>
      <td>https://en.wikipedia.org/wiki/Abdul_Latif_(cri...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ahmednagar</td>
      <td>Sadashiv Amrapurkar</td>
      <td>925429</td>
      <td>Abdul Latif was an underworld figure in Gujara...</td>
      <td>https://en.wikipedia.org/wiki/Abdul_Latif_(cri...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aizawl</td>
      <td>Zohmingliana Ralte</td>
      <td>24268</td>
      <td>Zohmingliana Ralte (born 2 October 1990) is an...</td>
      <td>https://en.wikipedia.org/wiki/Zohmingliana_Ralte</td>
    </tr>
  </tbody>
</table>
</div>



There seem to be something weird with certain cells having values from the row above them for Description and Links. Let's just fix them since rerunning the code will take too damn long. This way if a person is associated with two cities, their entries won't be deleted.


```python
y = []
for i in range(len(data.duplicated(subset ='Description'))):
    if(data.duplicated(subset ='Description')[i]==True):
        y.append(i)
for i,j in data.iterrows():
    if i in y:
        data.at[i, "Description"] = wikipedia.summary(j[1])
        data.at[i, "link"]= wikipedia.page(j[1]).url
```

### Scraping Left
The following link is to a page which has a list of 30 more pages which have lists of people from specific cities. Some are common with those extracted above and some aren't. I couldn't figure out a nice and automated way of scraping them so I'm not doing this for now. Perhaps some other time when I feel like it.


```python
link = "https://en.wikipedia.org/wiki/Category:Lists_of_people_by_city_in_India"

```

### City Coordinates
Now, let's get coordinates for each of the cities using the geopy module.


```python
### import geopy as gp
lats = []
longs = []
locator = gp.Nominatim(user_agent="apoorv")
from geopy.extra.rate_limiter import RateLimiter
geocode = RateLimiter(locator.geocode, min_delay_seconds=1)
from tqdm import tqdm
data['location'] = data['City'].apply(geocode)

data['point'] = data['location'].apply(lambda loc: tuple(loc.point) if loc else None)
# 4 - split point column into latitude, longitude and altitude columns
data[['latitude', 'longitude', 'altitude']] = pd.DataFrame(data['point'].tolist(), index=data.index)
```

Let's see what we got.


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
      <th>City</th>
      <th>Person</th>
      <th>Number of Views</th>
      <th>Description</th>
      <th>link</th>
      <th>location</th>
      <th>point</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>altitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Agartala</td>
      <td>Dipa Karmakar</td>
      <td>1345973</td>
      <td>Dipa Karmakar (born 9 August 1993) is an India...</td>
      <td>https://en.wikipedia.org/wiki/Dipa_Karmakar</td>
      <td>(Agartala, Mohanpur, West Tripura, Tripura, 79...</td>
      <td>(23.8312377, 91.2823821, 0.0)</td>
      <td>23.831238</td>
      <td>91.282382</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Agra</td>
      <td>Mumtaz Mahal</td>
      <td>2758109</td>
      <td>Mumtaz Mahal (Persian: ممتاز محل [mumˈt̪aːz mɛ...</td>
      <td>https://en.wikipedia.org/wiki/Mumtaz_Mahal</td>
      <td>(Agra, Uttar Pradesh, 280001, India, (27.17525...</td>
      <td>(27.1752554, 78.0098161, 0.0)</td>
      <td>27.175255</td>
      <td>78.009816</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ahmedabad</td>
      <td>Abdul Latif (criminal)</td>
      <td>2087052</td>
      <td>Abdul Latif was an underworld figure in Gujara...</td>
      <td>https://en.wikipedia.org/wiki/Abdul_Latif_(cri...</td>
      <td>(Ahmedabad, Ahmadabad City Taluka, Ahmedabad D...</td>
      <td>(23.0216238, 72.5797068, 0.0)</td>
      <td>23.021624</td>
      <td>72.579707</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ahmednagar</td>
      <td>Sadashiv Amrapurkar</td>
      <td>925429</td>
      <td>Sadashiv Dattaray Amrapurkar (11 May 1950 – 3 ...</td>
      <td>https://en.wikipedia.org/wiki/Sadashiv_Amrapurkar</td>
      <td>(Ahmednagar, Maharashtra, India, (19.162772500...</td>
      <td>(19.162772500000003, 74.85802430085195, 0.0)</td>
      <td>19.162773</td>
      <td>74.858024</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aizawl</td>
      <td>Zohmingliana Ralte</td>
      <td>24268</td>
      <td>Zohmingliana Ralte (born 2 October 1990) is an...</td>
      <td>https://en.wikipedia.org/wiki/Zohmingliana_Ralte</td>
      <td>(Aizawl, Tlangnuam, Aizwal, Mizoram, 796190, I...</td>
      <td>(23.7414092, 92.7209297, 0.0)</td>
      <td>23.741409</td>
      <td>92.720930</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



Remove unnecessary columns of point, location and altitude and export.


```python
data = data.drop(columns = ['point', 'location', 'altitude'])
data.to_csv("F:/Playin/PeopleMap.csv")
```

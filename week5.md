Introduction to Business Problem:

Opening a new Italian Restaurant in Bangalore, Karnataka
The objective of this report is to determine the best possible location to open an Italian Restaurant in Bangalore, Karnataka based on the different localities of the city, already established Italian restaurant in varios geographical location and ease of accessibility by maximum number of people so that the revenue from the latest venture can be maximized.
Data
T
his project will use data from :

Geopy - For getting the co-ordinated of different locations.
Foursquare API - To get the list of vanues and their details around a given location.
Methodology
Getting the co-ordinates of the target city.
Getting the list of neighborhoods and their co-ordinates.
Exploring the most visited venues in the target localities.
Clustering the localities.
Analyzing the clusters formed.
Collecting information about the type of restaurants already present in a locality.
Creating a machine learning model based on data acquired.
Using the model to predict the locality best suited for upcoming Italian Resataurant.


```python
#Importing required libraries
import numpy as np
import pandas as pd

from geopy.geocoders import Nominatim
try:
    import geocoder
except:
    !pip install geocoder
    import geocoder

import requests
from bs4 import BeautifulSoup

try:
    import folium
except:
    !pip install folium
    import folium
    
from sklearn.cluster import KMeans

from sklearn import preprocessing

from sklearn.model_selection import train_test_split

from sklearn.neighbors import KNeighborsClassifier

from sklearn import metrics

import matplotlib as mpl
import matplotlib.pyplot as plt
```


```python
!pip install wordcloud
# import package and its set of stopwords
from wordcloud import WordCloud, STOPWORDS

print ('Wordcloud is installed and imported!')
```

    Collecting wordcloud
      Downloading wordcloud-1.8.1-cp38-cp38-win_amd64.whl (155 kB)
    Requirement already satisfied: matplotlib in c:\users\yeshw\anaconda3\lib\site-packages (from wordcloud) (3.3.2)
    Requirement already satisfied: pillow in c:\users\yeshw\anaconda3\lib\site-packages (from wordcloud) (8.0.1)
    Requirement already satisfied: numpy>=1.6.1 in c:\users\yeshw\anaconda3\lib\site-packages (from wordcloud) (1.19.2)
    Requirement already satisfied: certifi>=2020.06.20 in c:\users\yeshw\anaconda3\lib\site-packages (from matplotlib->wordcloud) (2020.6.20)
    Requirement already satisfied: kiwisolver>=1.0.1 in c:\users\yeshw\anaconda3\lib\site-packages (from matplotlib->wordcloud) (1.3.0)
    Requirement already satisfied: cycler>=0.10 in c:\users\yeshw\anaconda3\lib\site-packages (from matplotlib->wordcloud) (0.10.0)
    Requirement already satisfied: pyparsing!=2.0.4,!=2.1.2,!=2.1.6,>=2.0.3 in c:\users\yeshw\anaconda3\lib\site-packages (from matplotlib->wordcloud) (2.4.7)
    Requirement already satisfied: python-dateutil>=2.1 in c:\users\yeshw\anaconda3\lib\site-packages (from matplotlib->wordcloud) (2.8.1)
    Requirement already satisfied: six in c:\users\yeshw\anaconda3\lib\site-packages (from cycler>=0.10->matplotlib->wordcloud) (1.15.0)
    Installing collected packages: wordcloud
    Successfully installed wordcloud-1.8.1
    Wordcloud is installed and imported!
    


```python
#Getting the location of Bangalore city using the geocoder package
g = geocoder.arcgis('Bangalore, India')
blr_lat = g.latlng[0]
blr_lng = g.latlng[1]
print("The Latitude and Longitude of Bangalore is {} and {}".format(blr_lat, blr_lng))
```

    The Latitude and Longitude of Bangalore is 12.966180000000065 and 77.58690000000007
    


```python
#Scraping the Wikimedia webpage for list of localities present in Bangalore city
neig = requests.get("https://commons.wikimedia.org/wiki/Category:Suburbs_of_Bangalore").text
```


```python
#parsing the scraped content
soup = BeautifulSoup(neig, 'html.parser')
```


```python
#Creating a list to store neighborhood data
neighborhoodlist = []
```


```python
#Searching the localities using class labels and appending it to the neighborhood list
for i in soup.find_all('div', class_='mw-category')[0].find_all('a'):
    neighborhoodlist.append(i.text)

#Creating a dataframe from the list
neig_df = pd.DataFrame({"Locality": neighborhoodlist})
neig_df.head()
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
      <th>Locality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Agara, Bangalore</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Arekere</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Banashankari</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Banaswadi</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Basavanagudi</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Shape of dataframe neig_df
neig_df.shape
```




    (58, 1)




```python
#Defining a function to get the location of the localities
def get_location(localities):
    g = geocoder.arcgis('{}, Bangalore, India'.format(localities))
    get_latlng = g.latlng
    return get_latlng
```


```python
#Creating an empty list
co_ordinates = []
#Getting the co-ordinates of each locality using the function defined above
for i in neig_df["Locality"].tolist():
    co_ordinates.append(get_location(i))
print(co_ordinates)
```

    [[12.842830000000049, 77.48759000000007], [12.885680000000036, 77.59668000000005], [12.922310000000039, 77.56988000000007], [13.019526632499165, 77.65502770568062], [12.938980000000072, 77.57137000000006], [12.882450000000063, 77.62475000000006], [12.927340000000072, 77.67169000000007], [12.966180000000065, 77.58690000000007], [12.966180000000065, 77.58690000000007], [12.817530000000033, 77.67879000000005], [12.966234036390167, 77.60681984212124], [12.794010000000071, 77.70015000000006], [13.250110000000063, 77.70788000000005], [12.966180000000065, 77.58690000000007], [12.943290000000047, 77.65602000000007], [12.845470000000034, 77.66430000000008], [12.998940000000061, 77.61276000000004], [12.942790000000059, 77.54122000000007], [13.02642000000003, 77.62437000000006], [13.049810000000036, 77.58903000000004], [13.02859334367402, 77.72464962552914], [12.912220000000048, 77.64470000000006], [13.030060000000049, 77.49526000000003], [12.923440000000028, 77.54284000000007], [12.908310000000029, 77.59024000000005], [13.075640000000021, 77.60394000000008], [12.928720000000055, 77.58281000000005], [12.96601000000004, 77.65767000000005], [12.906700000000058, 77.40467000000007], [12.966200000000072, 77.64982000000003], [12.882330000000024, 77.56926000000004], [12.920040000000029, 77.62546000000003], [13.000390000000039, 77.68368000000004], [12.967520000000036, 77.71500000000003], [12.920520000000067, 77.62090000000006], [12.99687504442263, 77.44306506935649], [12.994090000000028, 77.66633000000007], [12.977590000000077, 77.57256000000007], [13.006319846827683, 77.5684048532483], [12.954660000000047, 77.70752000000005], [13.032350000000065, 77.55866000000003], [12.955650000000048, 77.65335000000005], [12.956240000000037, 77.50936000000007], [13.005440000000021, 77.55693000000008], [12.93178000000006, 77.52668000000006], [13.023820000000057, 77.67785000000003], [13.062710000000038, 77.58550000000008], [13.113200000000063, 77.42463000000004], [12.98720000000003, 77.60401000000007], [12.953500000000076, 77.72114000000005], [12.971920000000068, 77.64778000000007], [12.971380000000067, 77.59582000000006], [12.989080000000058, 77.62795000000006], [12.943480000000022, 77.74703000000005], [12.979380000000049, 77.73372000000006], [13.09931000000006, 77.59259000000003], [13.039120000000025, 77.57797000000005], [13.029550000000029, 77.54022000000003]]
    


```python
co_ordinates[:5]
```




    [[12.842830000000049, 77.48759000000007],
     [12.885680000000036, 77.59668000000005],
     [12.922310000000039, 77.56988000000007],
     [13.019526632499165, 77.65502770568062],
     [12.938980000000072, 77.57137000000006]]




```python
#Creating a dataframe from the list of location co-ordinates
co_ordinates_df = pd.DataFrame(co_ordinates, columns=['Latitudes', 'Longitudes'])
```


```python
#Adding co-ordinates of localities to neig_df dataframe
neig_df["Latitudes"] = co_ordinates_df["Latitudes"]
neig_df["Longitudes"] = co_ordinates_df["Longitudes"]
```


```python
print("The shape of neig_df is {}".format(neig_df.shape))
neig_df.head()
```

    The shape of neig_df is (58, 3)
    




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
      <th>Locality</th>
      <th>Latitudes</th>
      <th>Longitudes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Agara, Bangalore</td>
      <td>12.842830</td>
      <td>77.487590</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Arekere</td>
      <td>12.885680</td>
      <td>77.596680</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Banashankari</td>
      <td>12.922310</td>
      <td>77.569880</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Banaswadi</td>
      <td>13.019527</td>
      <td>77.655028</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Basavanagudi</td>
      <td>12.938980</td>
      <td>77.571370</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Creating a map
blr_map = folium.Map(location=[blr_lat, blr_lng],zoom_start=11)

#adding markers to the map for localities
#marker for Bangalore
folium.Marker([blr_lat, blr_lng], popup='<i>Bangalore</i>', color='red', tooltip="Click to see").add_to(blr_map)

#markers for localities
for latitude,longitude,name in zip(neig_df["Latitudes"], neig_df["Longitudes"], neig_df["Locality"]):
    folium.CircleMarker(
        [latitude, longitude],
        radius=6,
        color='blue',
        popup=name,
        fill=True,
        fill_color='#3186ff'
    ).add_to(blr_map)

blr_map


```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-31-7f6771e2e442> in <module>
          4 #adding markers to the map for localities
          5 #marker for Bangalore
    ----> 6 folium.Marker([blr_lat, blr_lng], popup='<i>Bangalore</i>', color='red', tooltip="Click to see").add_to(blr_map)
          7 
          8 #markers for localities
    

    TypeError: __init__() got an unexpected keyword argument 'color'



```python
#Plot the cluster on map
cluster_map = folium.Map(location=[blr_lat, blr_lng],zoom_start=11)

#marker for Bangalore
folium.Marker([blr_lat, blr_lng], popup='<i>Bangalore</i>', color='red', tooltip="Click to see").add_to(cluster_map)

#predicted locality
pred_locality = None
for locality, prediction in zip(neig_df_with_sv_interm['Locality'], neig_df_with_sv_interm['Prediction']):
    if prediction == "Italian Restaurant":
        pred_locality = locality

#Getting the colors for the clusters
col = ['red', 'green', 'blue']

#markers for localities
for latitude,longitude,name,clus in zip(blr_labels["Latitudes"], blr_labels["Longitudes"], blr_labels["Locality"], blr_labels["Cluster Label"]):
    label = folium.Popup(name + ' - Cluster ' + str(clus+1))
    if name==pred_locality:
        folium.Marker([latitude, longitude], popup=name, color='orange', tooltip="This is the predicted locality for opening a new Italian Restaurant.").add_to(cluster_map)
    else:
        folium.CircleMarker(
            [latitude, longitude],
            radius=6,
            color=col[clus],
            popup=label,
            fill=False,
            fill_color=col[clus],
            fill_opacity=0.3
        ).add_to(cluster_map)
       
cluster_map
```

Conclusion:
From above analysis we can infer that cluster-1(shown with red color) has no existing Italian Restaurant with the highest numbers of the same in cluster-2(shown with green color) and moderate number of Italian Restaurants are present in cluster-3(shown with blue color) located in the central part of the city.
This analysis presents a great opportunity to entrepreneurs to tap into the unutilized potential of the outer parts of the city of Bangalore by opening Italian Restaurants.
It is also evident that cluster-2(around the central part of the city) is suffering from high competition and over supply, hence investment in this area should be avoided by developers.
Developers with unique selling propositions that can stand out from the moderate competiton in cluste-3 and can take moderate risk and attract the customers already visiting the locality of this cluster because of the existing Italian Restaurant.
Results for the Machine Learning Model Our model has been trained, based on 30 other venues present in a locality which has atleast one Italian restaurant. We then also collected 30 venues from each locality that does not contain any Italian restaurant. The model then predicts the best locality by matching the combination of venues present in other localities that is already sustaining other Italian restaurant. The outcome from the Machine learning suggests that "UB City" Locality in Bangalore can be one of the suitable location for opening a New Italian Restaurant. Though, Thubarahalli belongs to Cluster-3(shown in blue color) which medium density of existing Italian restaurant, the combination of already present venues will be able to attract customer and maintain footfall, as done in other locations with similar composition of venues. Also by choosing a locality in Cluster-3, we can avoid un-neccessary competition already present in Cluster-2 and scarcity of footfall in Cluster-

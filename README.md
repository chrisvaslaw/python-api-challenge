# WeatherPy
----

#### Note
* Instructions have been included for each segment. You do not have to follow them exactly, but they are included to help you think through the steps.

# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import time
from scipy.stats import linregress
import urllib
from sklearn.linear_model import LinearRegression
import scipy.stats as st

# Import API key
from api_keys import weather_api_key

# Incorporated citipy to determine city based on latitude and longitude
from citipy import citipy

# Output File (CSV)
output_data_file = "resources/cities.csv"

# Range of latitudes and longitudes
lat_range = (-90, 90)
lng_range = (-180, 180)

## Generate Cities List

# List for holding lat_lngs and cities
lat_lngs = []
cities = []

# Create a set of random lat and lng combinations
lats = np.random.uniform(lat_range[0], lat_range[1], size=1500)
lngs = np.random.uniform(lng_range[0], lng_range[1], size=1500)
lat_lngs = zip(lats, lngs)

# Identify nearest city for each lat, lng combination
for lat_lng in lat_lngs:
    city = citipy.nearest_city(lat_lng[0], lat_lng[1]).city_name
    
    # If the city is unique, then add it to a our cities list
    if city not in cities:
        cities.append(city)

# Print the city count to confirm sufficient count
len(cities)

### Perform API Calls
* Perform a weather check on each city using a series of successive API calls.
* Include a print log of each city as it'sbeing processed (with the city number and city name).


# Build API Call URL
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "metric"
query_url = f"{url}appid={weather_api_key}&units={units}&q="
query_url;

# Print Introduction
print("Begin Data Retrieval         ")
print("-----------------------------")

# Create empty list to hold city data
city_data = []

# Create counters
records = 1
sets = 1

# Loop through all the cities in our list
for i, city in enumerate(cities,1):
        
# Group cities 
    if (i % 50 == 0 and i >= 50):
        sets += 1
        records = 0

# Create City URL 
    city_url = query_url + urllib.request.pathname2url(city)

# Log the url, record, and set numbers
    print("Processing Record %s of Set %s | %s" % (records, sets, city))
    print(city_url)

# Add 1 to the record count
    records += 1

# Run an API request for each of the cities
    try:
        
# Retrieve the Data
        city_weather = requests.get(city_url).json()

# Get Data Stats
        city_latitute = city_weather["coord"]["lat"]
        city_longitude = city_weather["coord"]["lon"]
        city_max_temperature = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
        city_date = city_weather["dt"]

# Append the City information into city_data list
        city_data.append({"City": city, 
                          "Lat": city_latitute, 
                          "Lng": city_longitude, 
                          "Max Temp": city_max_temperature,
                          "Humidity": city_humidity,
                          "Cloudiness": city_clouds,
                          "Wind Speed": city_wind,
                          "Country": city_country,
                          "Date": city_date})

# If an error is experienced, skip the city
    except:
        print("City not found...")
        pass
              
# Indicate that Data Loading is complete 
print("-----------------------------")
print("End Data Retrieval           ")

### Convert Raw Data to DataFrame
* Export the city data into a .csv.
* Display the DataFrame

# Create Data Frame
weather_data_pd = pd.DataFrame(city_data)

# Create CSV
weather_data_pd.to_csv("WeatherPy.csv",encoding="utf-8",index=False)

# Display Data
weather_data_pd.head()

# Look for empty rows
weather_data_pd.count()

# Find Cities with Humidity over 100
humidity_df = weather_data_pd.loc[(weather_data_pd["Humidity"] > 100)]
humidity_df

# Make a new DataFrame equal to the city data to drop all humidity outliers by index.
weather_data_df = weather_data_pd.loc[(weather_data_pd["Humidity"] < 100)]

# Sort Humidity Values
weather_data_df.sort_values('Humidity', ascending = False, inplace = True)
weather_data_df

## Plotting the Data
* Use proper labeling of the plots using plot titles (including date of analysis) and axes labels.
* Save the plotted figures as .pngs.

## Latitude vs. Temperature Plot

plt.scatter(weather_data_df['Lat'],weather_data_df['Max Temp'])
plt.xlabel("Latitude")
plt.ylabel("Temperature")
plt.show()

## Latitude vs. Humidity Plot

plt.scatter(weather_data_df['Lat'],weather_data_df['Humidity'])
plt.xlabel("Latitude")
plt.ylabel("Humidity")
plt.show()

## Latitude vs. Cloudiness Plot

plt.scatter(weather_data_df['Lat'],weather_data_df['Cloudiness'])
plt.xlabel("Latitude")
plt.ylabel("Cloudiness")
plt.show()

## Latitude vs. Wind Speed Plot

plt.scatter(weather_data_df['Lat'],weather_data_df['Wind Speed'])
plt.xlabel("Latitude")
plt.ylabel("Wind Speed")
plt.show()

## Linear Regression

# Separate cities in Northern and Southern Hemispheres
northern = weather_data_df.loc[weather_data_df['Lat'] >= 0]
southern = weather_data_df.loc[weather_data_df['Lat'] < 0]

# Print Northern Data Frame
northern

# Print Southern Data Frame
southern

####  Northern Hemisphere - Max Temp vs. Latitude Linear Regression

lat = northern.iloc[:,1]
temp = northern.iloc[:,3]
correlation = st.pearsonr(lat,temp)
print(f"The correlation between latitude and temperature in the Northern Hemisphere is {round(correlation[0],2)}")

####  Southern Hemisphere - Max Temp vs. Latitude Linear Regression

lat = southern.iloc[:,1]
temp = southern.iloc[:,3]
correlation = st.pearsonr(lat,temp)
print(f"The correlation between latitude and temperature in the Southern Hemisphere is {round(correlation[0],2)}")

####  Northern Hemisphere - Humidity (%) vs. Latitude Linear Regression

lat = northern.iloc[:,1]
hum = northern.iloc[:,4]
correlation = st.pearsonr(lat,hum)
print(f"The correlation between latitude and humidity in the Northern Hemisphere is {round(correlation[0],2)}")

####  Southern Hemisphere - Humidity (%) vs. Latitude Linear Regression

lat = southern.iloc[:,1]
hum = southern.iloc[:,4]
correlation = st.pearsonr(lat,hum)
print(f"The correlation between latitude and humidity in the Southern Hemisphere is {round(correlation[0],2)}")

####  Northern Hemisphere - Cloudiness (%) vs. Latitude Linear Regression

lat = northern.iloc[:,1]
cld = northern.iloc[:,5]
correlation = st.pearsonr(lat,cld)
print(f"The correlation between latitude and cloudiness in the Northern Hemisphere is {round(correlation[0],2)}")

####  Southern Hemisphere - Cloudiness (%) vs. Latitude Linear Regression

lat = southern.iloc[:,1]
cld = southern.iloc[:,5]
correlation = st.pearsonr(lat,cld)
print(f"The correlation between latitude and cloudiness in the Southern Hemisphere is {round(correlation[0],2)}")

####  Northern Hemisphere - Wind Speed (mph) vs. Latitude Linear Regression

lat = northern.iloc[:,1]
wind = northern.iloc[:,6]
correlation = st.pearsonr(lat,wind)
print(f"The correlation between latitude and wind speed in the Northern Hemisphere is {round(correlation[0],2)}")

####  Southern Hemisphere - Wind Speed (mph) vs. Latitude Linear Regression

lat = southern.iloc[:,1]
wind = southern.iloc[:,6]
correlation = st.pearsonr(lat,wind)
print(f"The correlation between latitude and wind speed in the Southern Hemisphere is {round(correlation[0],2)}")

_________________________________________________________________________________________________________________________________________________________________________________

# VacationPy
----

#### Note
* Keep an eye on your API usage. Use https://developers.google.com/maps/reporting/gmp-reporting as reference for how to monitor your usage and billing.

* Instructions have been included for each segment. You do not have to follow them exactly, but they are included to help you think through the steps.

# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import gmaps
import os
import json 
# Import API key
from api_keys import g_key

# !pip install gmaps;
# !jupyter nbextension enable --py gmaps;
# pd.set_option('display.max_rows', None)

### Store Part I results into DataFrame
* Load the csv exported in Part I to a DataFrame

weather_file = "Resources/WeatherPy.csv"
weather_file_df = pd.read_csv(weather_file)
weather_file_df

### Humidity Heatmap
* Configure gmaps.
* Use the Lat and Lng as locations and Humidity as the weight.
* Add Heatmap layer to map.

 # Configure gmaps with API key
gmaps.configure(api_key=g_key)

# Store 'Lat' and 'Lng' into  locations 
locations = weather_file_df[["Lat", "Lng"]].astype(float)

# Convert Humidity to float and store
humidity = weather_file_df["Humidity"].astype(float)

# Create a Humidity Heatmap layer
fig = gmaps.figure()

heat_layer = gmaps.heatmap_layer(locations, weights=humidity, 
                                 dissipating=False, max_intensity=humidity.max(),
                                 point_radius = 1)

fig.add_layer(heat_layer)

fig

### Create new DataFrame fitting weather criteria
* Narrow down the cities to fit weather conditions.
* Drop any rows will null values.

# Convert Celsius to Fahrenheit 
weather_file_df["Temp (far)"] = (weather_file_df["Max Temp"] * 1.8) + 32

# Use LOC to narrow Data Frame for cities with great weather
good_weather = weather_file_df.loc[(weather_file_df["Temp (far)"] > 70) & 
                                   (weather_file_df["Temp (far)"] < 80) & 
                                   (weather_file_df["Cloudiness"] < 10), :] 
good_weather

### Hotel Map
* Store into variable named `hotel_df`.
* Add a "Hotel Name" column to the DataFrame.
* Set parameters to search for hotels with 5000 meters.
* Hit the Google Places API for each city's coordinates.
* Store the first Hotel result into the DataFrame.
* Plot markers on top of the heatmap.

# Create empty hotel list
hotels = []

# Loop through the index values in good_weather df and isolate for lat and long for new API call
for index in good_weather.index.values: 
    lat = good_weather.loc[index]['Lat']
    lng = good_weather.loc[index]['Lng']
    
# Set parameters for g-map call
    parameters = {
        "location": f"{lat},{lng}",
        "radius": 5000,
        "types" : "hotel",
        "key": g_key
                 }
# Build URl for API Call
    url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    hotel_search = requests.get(url, parameters).json()
    # print(response.url)
    # print(json.dumps(hotel_search, indent=4, sort_keys=True))

# Append hotels list to add first hotel name from our hotel loop
    try:
        hotels.append(hotel_search['results'][1]['name'])

# Create an entry in case no hotels are found
    except:
        hotels.append("No hotel found")

# Add a column to hold to the Data Frame to hold the hotel list
good_weather["Hotel Name"] = hotels
hotel_df = good_weather
hotel_df

# NOTE: Do not change any of the code in this cell

# Using the template add the hotel marks to the heatmap
info_box_template = """
<dl>
<dt>Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
</dl>
"""
# Store the DataFrame Row
# NOTE: be sure to update with your DataFrame name
hotel_info = [info_box_template.format(**row) for index, row in hotel_df.iterrows()]
locations = hotel_df[["Lat", "Lng"]]

hotel_df["Hotel Info"] = hotel_info
hotel_df

# Add marker layer ontop of heat map
markers = gmaps.marker_layer(locations)
fig.add_layer(markers)

# Display figure
fig

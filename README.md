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
# Import API key
from api_keys import g_key

# !pip install gmaps;
# !jupyter nbextension enable --py gmaps
!jupyter labextension install @jupyter-widgets/jupyterlab-manager
# !jupyter nbextension enable --py widgetsnbextension


### Store Part I results into DataFrame
* Load the csv exported in Part I to a DataFrame

weather_file = "WeatherPy.csv"
weather_file_df = pd.read_csv(weather_file)
weather_file_df.head(100)

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
                                 point_radius = 3)

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
# Loop through the latitutde and longtitue columns for g_map API call parameter input
for index in good_weather.index.values:
    lat = good_weather.loc[index]['Lat']
    lng = good_weather.loc[index]['Lng']
    
# Set parameters for g-map call

    parameters = {
        "location": f"{lat},{lng}",
        "radius": 5000,
        "type" : "hotel",
        "key": g_key
                    }
# Build URl for API Call 
    url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    query_url = requests.get(url, parameters)
    
# Use Json to view call results
    json = query_url.json()
    
# Set up a variable to hold json results 
    hotel = json['results'][0]['name']
    print(json.dumps(seattle_bikes, indent=4, sort_keys=True))
    try:
        hotels.append(hotel)
        #good_weather.loc[index]['Hotel Name'] = hotel
    except:
        hotels.append("")
        #good_weather.loc[index]['Hotel Name'] = ""

# print(hotels)
#good_weather["Hotel Name"] = pd.Series(hotels)
#good_weather =good_weather.dropna(how='any')
# good_weather



a

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

# Add marker layer ontop of heat map


# Display figure



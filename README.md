# World_Weather_Analysis

The purpose of this project is to build the code for an app where customers can use randomly generated locations to search for ideal vacation spots based on their ideal weather conditions.  Building upon that further, this project also uses APIs to create an itinerary and route between cities. 

## Project Overview

To start, I generated 2000 random longitudes and latitudes and used citipy to generate a unique list of cities close to those coordinates. Then, using the OpenWeather APIs to loop through the cities, pull back country code and weather data for each one, and generating a new DataFrame with the information.  After cleaning up the DataFrame, I saved it as a csv to be used for the next step of the project.

Next, I imported the csv, and created an input for users to plug in their minimum and maximum temperature requirements.  From there, we created a new DataFrame based on this weather data. In this case, we went down from a DataFrame with 715 cities to one with 286 cities.  I took this data and created a new DataFrame with a column to capture hotel information for these cities.  Using another for loop, we used the information within the DataFrame to search through the Geoapify APIs and pull back the first hotel for each city as is shown in the code below.

```
# Print a message to follow up the hotel search
print("Starting hotel search")

# Iterate through the hotel_df DataFrame
for index, row in hotel_df.iterrows():
    
    # Get latitude and longitude from DataFrame.
    latitude = row["Lat"]
    longitude = row["Lng"]
    
    # Add the current city's latitude and longitude to the params dictionary
    params["filter"] = f"circle:{longitude},{latitude},{radius}"
    params["bias"] = f"proximity:{longitude},{latitude}"
    
    # Set up the base URL for the Geoapify Places API.
    base_url = "https://api.geoapify.com/v2/places"

    # Make request and retrieve the JSON data by using the params dictionaty
    name_address = requests.get(base_url, params=params)
    
    # Convert the API response to JSON format
    name_address = name_address.json()
    
    # Get the first hotel from the results and store the name, if a hotel isn't found set the hotel name as "No hotel found".
    try:
        hotel_df.loc[index, "Hotel Name"] = name_address["features"][0]["properties"]["name"]
    except (KeyError, IndexError):
        # If no hotel is found, set the hotel name as "No hotel found".
        hotel_df.loc[index, "Hotel Name"] = "No hotel found."
        
    # Log the search results
    print(f"{hotel_df.loc[index, 'City']} - nearest hotel: {hotel_df.loc[index, 'Hotel Name']}")

# Display sample data
hotel_df
```

Removing any cities where hotels weren't found, we were left with a DataFrame of 181 locations for our customers which I saved as a csv for the next step.  I used GeoViews to generate a map of these points as well.

In the final stage of the process, I looked at the GeoViews map and selected a section where our customers could potentially travel between points.  From here, I pulled each city into an itinerary DataFrame and used the Geoapify Routing API to find a route between the cities and put this on a new map for our customers.

## Summary Results

In summary this tool provides a quick way to visualize information for customers who are planning trips.  It can show us a wide variety of choices where we can get quick information about the city and build a route for multiple locations.

![Map of Cities in Range](https://github.com/ChallahBack83/World_Weather_Analysis/blob/main/Vacation_Search/WeatherPy_vacation_map.png)

![Route Map](https://github.com/ChallahBack83/World_Weather_Analysis/blob/main/Vacation_Itinerary/WeatherPy_travel_map.png)

---
layout: post
title:      "Module 1 Project EDA and Feature Addition"
date:       2019-09-26 20:34:59 +0000
permalink:  module_1_project_eda_and_feature_addition
---

## Project EDA

It's exciting that the first project is finally here! I'll admit it was pretty stressful to look at all the requirements and get started, but once I began the EDA I realized, "Hey I got this!" And while my analysis and model wasn't perfect I found the whole process very educational and easily one of the my favorite parts of the course so far since I got to really get my hands dirty with minimal hand holding. 

I began my EDA with reading in the King county housing data with pandas `pd.read_csv()` function and using `df.head()`  to see the first 5 rows. Pandas showed me a snippet of the dataframe and everything look liked it loaded ok. I proceeded to use `df.info()` to get column titles and number of entries. Most of the column titles looked normal given the context of the dataset. `View, grade, sqft_living15, and sqft_lot15 ` made me scratch my head a bit, so I consulted the original repo for context on these columns. Luckily there was a markdown file with context for each column. However, at least in the case of view, it didn't seem to quite matchup with the description. The description just states, "has been viewed," yet values ranged from 0-4 and not a number. I figured then that perhaps view indicated number of views based on the cardinal directions. Since I imagined most people wouldn't buy a house sight unseen. I decided to just convert all missing values to 0 since there were only 63. Waterfront and year renovated were the only other two missing values. I used `df.column_in_question.unique()` to see what unique values we had for each column. Waterfront had 0, 1, and not a number, while year renovated had 0, various years, and not a number. I changed all the of not a number values to zero. It didn't seem to make sense to use a mean or median for waterfront since it was a binary category, either true or false. It also didn't make sense to do change not several values to anything, but 0, since it was either renovated or a certain year or it wasn't. I could have also removed some of the outliers like 33 bedrooms and the 13,000 sq. ft property. Considering the number of entries overall, I thought they wouldn't have a huge effect on my future modeling. Given time to improve my model I'd probably remove them since they can have a big effect on any formula that uses the mean. 

I also made a [histogram](https://drive.google.com/file/d/1ZkpfdyQOCIZej2exy0DxStbf7SSbDzAg/view?usp=sharing) of each feature to look for normal distributions in each column. Nearly all continuous features looked fairly normal with some positive skew except distance from Amazon. In retrospect I probably should have scaled most of my features to help with my modeling. 

## Feature addition


Next I decided to add some features. I got the idea and code for adding price per square foot from [whitcrrd](https://github.com/whitcrrd/kc_housing_linear_regression), who was in a previous cohort. Here's the code in question:

`kings_data['price_per_sqft'] = kings_data['price'] / kings_data['sqft_living']`.

It made sense to add because it's a common metric when purchasing a house or even renting. A buyer typically wants to know how much house they're getting for their money and a seller can potentially greatly increase their sale price if can add more square feet to home with a high price per square foot. 

I also decided to add a feature called distance from Amazon. Amazon HQ employs nearly 54,000 employees in the Seattle metro as of [2019](https://www.geekwire.com/2019/amazon-surpasses-microsoft-number-seattle-region-employees-amid-big-growth-plans-across-us/). They also pay very well. As a south bay native, I've seen lots of companies come into the area and drive up home prices, like, Google and Facebook. Not to mention older companies like Apple and Cisco. 

To add this feature, I used a package called [geopy](https://geopy.readthedocs.io/en/stable/). It allows you to use a variety of geocoders to get information like latitude, longitude, altitude, address, and distance from another point. 

```
from geopy import distance
from geopy.geocoders import Nominatim

geolocator = Nominatim(user_agent="python project") # need to give geolocation a user_agent other than default
location = geolocator.geocode("410 Terry Ave. North, Seattle, WA") #Amazon address
Amazon = location.latitude, location.longitude #make a tupple of lat and long

kings_data['lat'] = kings_data['lat'].astype(str)
kings_data['long'] = kings_data['long'].astype(str)
kings_data['distance_from_Amazon'] = "" #create empty column 
```

You create a Nominatim object with a user agent string. It can basically be anything other than the default. In this case I got the Amazon main address and used `geolocator.gecode()` to get the latitude and longitude. I saved this to a tuple called Amazon. Each value is a string. I converted the existing latitude and longitude columns to strings. 

```
#use distance from geopy to get geodesic distance between the two locations
#and add to column 
for i, j in kings_data.iterrows():
    second_location = (j.lat, j.long)
    x = distance.distance(Amazon, second_location).miles
    kings_data.loc[[0, i], 'distance_from_Amazon'] = x
#convert values to float since it defaults to string    
kings_data['distance_from_Amazon'] = kings_data['distance_from_Amazon'].astype(np.float64)
```

Next I used `df.iterrows()` to loop through my rows and get the distance by feeding a tuple with the current row's latitude and longitude along with the previously defined Amazon one. The `distance.distance()` function gets the geodesic distance between two locations and returns that as a string in kilometers by default. Appending `.miles` converts it to miles as shown. Then I converted all the values to a float to make it easier for [visualization](https://drive.google.com/file/d/1335GTlcY8cXuehCuEH-g6OlOMYxQomBD/view?usp=sharing) and modeling. There may have been an easier way to go about this, but overall, I'm happy with how it turned out.


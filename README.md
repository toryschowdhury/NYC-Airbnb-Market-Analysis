Introduction
Welcome to the NYC Airbnb market analysis! In this project, we will explore various datasets related to Airbnb listings in New York City. Our goal is to clean and analyze the data to answer key questions about pricing, room types, and market comparisons.

Datasets Used
airbnb_price.csv - Contains pricing information for Airbnb listings. airbnb_room_type.xlsx - Contains details about the types of rooms available. airbnb_last_review.tsv - Contains information about the last reviews for each listing.

1. Importing the Data
We will start by importing the necessary libraries and loading our datasets.

import pandas as pd
import numpy as np
import datetime as dt

# Load airbnb_price.csv
prices = pd.read_csv("airbnb_price.csv")

print(f"Prices:\n{prices.head()}\n")
Prices:
   listing_id        price                nbhood_full
0        2595  225 dollars         Manhattan, Midtown
1        3831   89 dollars     Brooklyn, Clinton Hill
2        5099  200 dollars     Manhattan, Murray Hill
3        5178   79 dollars  Manhattan, Hell's Kitchen
4        5238  150 dollars       Manhattan, Chinatown

import pandas as pd

# Load the Excel file using a specific engine
room_types = pd.read_excel("airbnb_room_type.xlsx")
# Load airbnb_last_review.tsv
reviews = pd.read_csv("airbnb_last_review.tsv", sep="\t")
print(f"Reviews:\n{reviews.head()}\n")
Reviews:
   listing_id    host_name   last_review
0        2595     Jennifer   May 21 2019
1        3831  LisaRoxanne  July 05 2019
2        5099        Chris  June 22 2019
3        5178     Shunichi  June 24 2019
4        5238          Ben  June 09 2019

2. Cleaning the Price Column
Next, we need to clean the price data to convert it into a numeric format.

# Remove ' dollars' from the price column and convert to numeric
prices["price"] = prices["price"].str.replace(" dollars", "").astype(float)

# Display the first five rows of the cleaned price column
print(prices["price"].head())

# Print descriptive statistics for the price column
print(prices["price"].describe())
0    225.0
1     89.0
2    200.0
3     79.0
4    150.0
Name: price, dtype: float64
count    25209.000000
mean       141.777936
std        147.349137
min          0.000000
25%         69.000000
50%        105.000000
75%        175.000000
max       7500.000000
Name: price, dtype: float64
3. Calculating Average Pric
calculate the average price per night for Airbnb listings in NYC.

# Remove listings with a price of $0
prices = prices[prices["price"] > 0]

# Calculate the average price
avg_price = round(prices["price"].mean(), 2)
print(f"The average price per night for an Airbnb listing in NYC is ${avg_price}.")
The average price per night for an Airbnb listing in NYC is $141.82.
4. Comparing Costs to the Private Rental Market
We will compare the average Airbnb price to the private rental market.

# Calculate monthly price from nightly price
prices["price_per_month"] = prices["price"] * 365 / 12
average_price_per_month = round(prices["price_per_month"].mean(), 2)

# Compare with private market price
private_market_price = 3100  # Zumper's average for a 1-bedroom apartment
print(f"Airbnb monthly costs are ${average_price_per_month}, while in the private market you would pay ${private_market_price}.")
Airbnb monthly costs are $4313.61, while in the private market you would pay $3100.
5. Cleaning the Room Type Column
Next, we will clean the room type data to ensure consistency.

# Convert room_type to lowercase and set as category
room_types["room_type"] = room_types["room_type"].str.lower().astype("category")

# Count the frequency of each room type
room_frequencies = room_types["room_type"].value_counts()
print(room_frequencies)
room_type
entire home/apt    13266
private room       11356
shared room          587
Name: count, dtype: int64
6. Analyzing Review Dates
We will analyze the review dates to find the earliest and latest reviews.

# Convert last_review to datetime
reviews["last_review"] = pd.to_datetime(reviews["last_review"])

# Find the earliest and latest review dates
first_reviewed = reviews["last_review"].min()
last_reviewed = reviews["last_review"].max()

print(f"The latest Airbnb review is {last_reviewed.date()}, the earliest review is {first_reviewed.date()}.")
The latest Airbnb review is 2019-07-09, the earliest review is 2019-01-01.
7. Joining the DataFrames
Now that we’ve extracted the necessary information, we will merge the three DataFrames to facilitate future analyses. After merging, we will remove any observations with missing values and check for duplicates.

# Merge prices and room_types to create rooms_and_prices
rooms_and_prices = pd.merge(prices, room_types, how="outer", on="listing_id")

# Merge rooms_and_prices with the reviews DataFrame to create airbnb_merged
airbnb_merged = pd.merge(rooms_and_prices, reviews, how="outer", on="listing_id")

# Drop missing values from airbnb_merged
airbnb_merged.dropna(inplace=True)

# Check for duplicate values
print("There are {} duplicates in the DataFrame.".format(airbnb_merged.duplicated().sum()))
There are 0 duplicates in the DataFrame.
8. Analyzing Listing Prices by NYC Borough
With all data combined into a single DataFrame, we will analyze the differences in listing prices across NYC boroughs. The boroughs are currently embedded in the nbhood_full column, so we will extract this information into a new column.

# Extract borough information from the nbhood_full column
airbnb_merged["borough"] = airbnb_merged["nbhood_full"].str.partition(",", expand=True)[0]

# Group by borough and calculate summary statistics
boroughs = airbnb_merged.groupby("borough")["price"].agg(["sum", "mean", "median", "count"])

# Round borough statistics to 2 decimal places and sort by mean price in descending order
boroughs = boroughs.round(2).sort_values("mean", ascending=False)

# Print borough statistics
print(boroughs)
                     sum    mean  median  count
borough                                        
Manhattan      1898417.0  184.04   149.0  10315
Brooklyn       1275250.0  122.02    95.0  10451
Queens          320715.0   92.83    70.0   3455
Staten Island    22974.0   86.04    71.0    267
Bronx            55156.0   79.25    65.0    696
9. Price Range by Borough
Next, we will categorize listings based on specific price ranges and view this by borough. We will create a new column, price_range, using defined categories.

# Define labels and ranges for price categories
label_names = ["Budget", "Average", "Expensive", "Extravagant"]
ranges = [0, 69, 175, 350, np.inf]

# Create a new column, price_range, using pd.cut to segment prices
airbnb_merged["price_range"] = pd.cut(airbnb_merged["price"], bins=ranges, labels=label_names)

# Calculate frequencies of price ranges by borough
prices_by_borough = airbnb_merged.groupby(["borough", "price_range"])["price_range"].agg("count")

# Print the frequency of listings in each price range by borough
print(prices_by_borough)
borough        price_range
Bronx          Budget          381
               Average         285
               Expensive        25
               Extravagant       5
Brooklyn       Budget         3194
               Average        5532
               Expensive      1466
               Extravagant     259
Manhattan      Budget         1148
               Average        5285
               Expensive      3072
               Extravagant     810
Queens         Budget         1631
               Average        1505
               Expensive       291
               Extravagant      28
Staten Island  Budget          124
               Average         123
               Expensive        20
               Extravagant       0
Name: price_range, dtype: int64
Visualizing the output:
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Assuming airbnb_merged DataFrame is already created and contains the necessary data

# Define labels and ranges for price categories
label_names = ["Budget", "Average", "Expensive", "Extravagant"]
ranges = [0, 69, 175, 350, np.inf]

# Create a new column, price_range, using pd.cut to segment prices
airbnb_merged["price_range"] = pd.cut(airbnb_merged["price"], bins=ranges, labels=label_names)

# Calculate frequencies of price ranges by borough
prices_by_borough = airbnb_merged.groupby(["borough", "price_range"])["price_range"].agg("count").unstack()

# Plotting the data
plt.figure(figsize=(12, 8))
prices_by_borough.plot(kind='bar', stacked=True)
plt.title('Frequency of Listings in Each Price Range by Borough')
plt.xlabel('Borough')
plt.ylabel('Number of Listings')
plt.legend(title='Price Range')
plt.xticks(rotation=45)
plt.tight_layout()

# Show the plot
plt.show()
<Figure size 1200x800 with 0 Axes>

import pandas as pd
import numpy as np
import plotly.express as px


# Define labels and ranges for price categories
label_names = ["Budget", "Average", "Expensive", "Extravagant"]
ranges = [0, 69, 175, 350, np.inf]

# Create a new column, price_range, using pd.cut to segment prices
airbnb_merged["price_range"] = pd.cut(airbnb_merged["price"], bins=ranges, labels=label_names)

# Calculate frequencies of price ranges by borough
prices_by_borough = airbnb_merged.groupby(["borough", "price_range"])["price_range"].agg("count").unstack().reset_index()

# Melt the DataFrame for Plotly
prices_by_borough_melted = prices_by_borough.melt(id_vars="borough", value_vars=label_names, var_name="Price Range", value_name="Count")

# Create an interactive plot using Plotly
fig = px.bar(prices_by_borough_melted, x="borough", y="Count", color="Price Range", 
             title="Frequency of Listings in Each Price Range by Borough",
             labels={"borough": "Borough", "Count": "Number of Listings"}, barmode="stack")

# Show the plot
fig.show()

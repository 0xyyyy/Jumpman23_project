# Jumpman_23 Analysis 
Jumpman23 is a new meal delivery service and is looking to grow their newest market in NYC. The following is an analysis of key growth areas to meet the 20% growth target. 

## Overview
Jumpman23 is a meal delivery service where jumpmen fulfill delivery orders for customers ordering from a variety of restaurants/stores. 

### Data Integrity Issues 
We begin by Loading data into a pandas dataframe for analysis 

```Python
#Load in csv 
df = pd.read_csv("analyze_me.csv")
```

We obtain info on the DataFrame columns and look for any data type issues and to get a better look at any columns that may have null values. 

```Python 
df.info()

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 5983 entries, 0 to 5982
Data columns (total 18 columns):
 #   Column                               Non-Null Count  Dtype  
---  ------                               --------------  -----  
 0   delivery_id                          5983 non-null   int64  
 1   customer_id                          5983 non-null   int64  
 2   jumpman_id                           5983 non-null   int64  
 3   vehicle_type                         5983 non-null   object 
 4   pickup_place                         5983 non-null   object 
 5   place_category                       5100 non-null   object 
 6   item_name                            4753 non-null   object 
 7   item_quantity                        4753 non-null   float64
 8   item_category_name                   4753 non-null   object 
 9   how_long_it_took_to_order            3038 non-null   object 
 10  pickup_lat                           5983 non-null   float64
 11  pickup_lon                           5983 non-null   float64
 12  dropoff_lat                          5983 non-null   float64
 13  dropoff_lon                          5983 non-null   float64
 14  when_the_delivery_started            5983 non-null   object 
 15  when_the_Jumpman_arrived_at_pickup   5433 non-null   object 
 16  when_the_Jumpman_left_pickup         5433 non-null   object 
 17  when_the_Jumpman_arrived_at_dropoff  5983 non-null   object 
dtypes: float64(5), int64(3), object(10)
memory usage: 841.5+ KB
```
-The time stamp objects are cast incorrectly. We will recast these as datetime objects for the 4 columns indicating Jumpman delivery time and to a timedelta object for 'how_long_it_took_to_order'. 

```Python 
new_df["how_long_it_took_to_order"] = pd.to_timedelta(new_df["how_long_it_took_to_order"])
new_df["when_the_delivery_started"] = pd.to_datetime(new_df["when_the_delivery_started"])
new_df["when_the_Jumpman_arrived_at_pickup"] = pd.to_datetime(new_df["when_the_Jumpman_arrived_at_pickup"])
new_df["when_the_Jumpman_left_pickup"] = pd.to_datetime(new_df["when_the_Jumpman_left_pickup"])
new_df["when_the_Jumpman_arrived_at_dropoff"] = pd.to_datetime(new_df["when_the_Jumpman_arrived_at_dropoff"])
```

-From this view we can also see that there are a number of rows with null values. This indicates a problem with data acquistion. Missing rows in the item_name, item_quantity, and item_category_name is a missed opportunity in regards to marketing campaigns for the NYC market. Without an understanding of customer purchasing habits it will be difficult to create targeted ad campaigns that could lead to increases in new customer acquisition. 

We then move on to checking for any duplicate values: 

```Python 
df.nunique()


delivery_id                            5214
customer_id                            3192
jumpman_id                              578
vehicle_type                              7
pickup_place                            898
place_category                           57
item_name                              2277
item_quantity                            11
item_category_name                      767
how_long_it_took_to_order              2579
pickup_lat                             1210
pickup_lon                             1179
dropoff_lat                            2841
dropoff_lon                            2839
when_the_delivery_started              5214
when_the_Jumpman_arrived_at_pickup     4719
when_the_Jumpman_left_pickup           4717
when_the_Jumpman_arrived_at_dropoff    5214
dtype: int64
```

The number of unique values for delivery_id are lower than the length of the dataframe, to investigate this issue further I check for any duplicate values in this column. 

```Python
pd.concat(g for _, g in df.groupby("delivery_id") if len(g) > 1)
df.loc[df['delivery_id'] == 1274248]
	delivery_id	customer_id	jumpman_id	vehicle_type	pickup_place	place_category	item_name	item_quantity	item_category_name	how_long_it_took_to_order	pickup_lat	pickup_lon	dropoff_lat	dropoff_lon	when_the_delivery_started	when_the_Jumpman_arrived_at_pickup	when_the_Jumpman_left_pickup	when_the_Jumpman_arrived_at_dropoff
2272	1274248	208020	60149	car	Murray's Falafel	Middle Eastern	Blue Lamoon Citrus blossom lemonade w/ Splenda	1.0	Beverages	00:07:08.767432	40.732166	-73.981904	40.747019	-73.990922	2014-10-01 17:25:48.54633	2014-10-01 17:40:32.886964	2014-10-01 17:53:54.166799	2014-10-01 18:09:37.353403
2299	1274248	208020	60149	car	Murray's Falafel	Middle Eastern	Moroccan Cigars (5 pc)	1.0	Appetizers	00:07:08.767432	40.732166	-73.981904	40.747019	-73.990922	2014-10-01 17:25:48.54633	2014-10-01 17:40:32.886964	2014-10-01 17:53:54.166799	2014-10-01 18:09:37.353403
2986	1274248	208020	60149	car	Murray's Falafel	Middle Eastern	Watermelon	1.0	Desserts	00:07:08.767432	40.732166	-73.981904	40.747019	-73.990922	2014-10-01 17:25:48.54633	2014-10-01 17:40:32.886964	2014-10-01 17:53:54.166799	2014-10-01 18:09:37.353403
```

It looks like different rows are generated for each item on the same delivery id. This could impact order volume analysis and show greater order volume for restaurants that receive multiple item requests per order. In order to ensure the integrity of the following analysis rows with duplicate delivery_id's will be expunged leaving a dataframe with one order per delivery_id. 

```Python 
new_df = df.drop_duplicates(subset="delivery_id")
if len(new_df) == df["delivery_id"].nunique():
    print("dataframe clean")
```

### Data Processing 

In addition to cleaning the DataFrame of duplicate rows we will create 4 new columns to aid in analysis: 
**prep_time:** Time between Jumpman arriving at pickup and leaving pickup. 
**transit_time:** Time between Leaivng pickup and arriving at drop off. 
**total_order_time:** Time from start of delivery to dropoff.
**distance_miles:** Distance traveled by Jumpman in miles.
```Python 
#Create column for prep time 
new_df["prep_time"] = new_df["when_the_Jumpman_left_pickup"] - new_df["when_the_Jumpman_arrived_at_pickup"]
#Convert time to minutes
new_df["prep_time"] = new_df["prep_time"] / np.timedelta64(1, 'm')
#Create column for transit time 
new_df["transit_time"] = new_df["when_the_Jumpman_arrived_at_dropoff"] - new_df["when_the_Jumpman_left_pickup"]
#convert time to minutes
new_df["transit_time"] = new_df["transit_time"] / np.timedelta64(1, 'm')
#Create Column for Total order time 
new_df["total_order_time"] = new_df["when_the_Jumpman_arrived_at_dropoff"] - new_df["when_the_delivery_started"]
#convert time to minutes
new_df["total_order_time"] = new_df["total_order_time"] / np.timedelta64(1, 'm')

#Determine distance traveled between pickup and dropoff, this returns distance traveled in miles
def haversine_distance(lat1, lon1, lat2, lon2):
   r = 6371
   phi1 = np.radians(lat1)
   phi2 = np.radians(lat2)
   delta_phi = np.radians(lat2 - lat1)
   delta_lambda = np.radians(lon2 - lon1)
   a = np.sin(delta_phi / 2)**2 + np.cos(phi1) * np.cos(phi2) *   np.sin(delta_lambda / 2)**2
   res = r * (2 * np.arctan2(np.sqrt(a), np.sqrt(1 - a)))
   return np.round(res, 2)*0.62137
   
#compute distance traveled in miles using haversine distance formula, round to 4 decimal places 
new_df["distance_miles"] = round(haversine_distance(new_df["pickup_lat"], new_df["pickup_lon"], new_df["dropoff_lat"], new_df["dropoff_lon"]),4)
```

### Data Analysis 

Based on the data, I believe a multi-pronged approach is required to achieve 20% market growth in NYC. 

**Data Acquisition and Integrity**

The number one priority to improving market growth in the near and long term future is improving data acquisition. Ensuring that we are able to capture all information regarding any order is crucial to gaining insights into customer behavior and spending patterns. Many of the following approaches will be aided by improvements to data acquisition. Specifically, being able to capture and gain insight on popular items is crucial to creating ad campaigns that are specific to the purchase habits of customers in the NYC market.

**Latency to Order Placement**

While investigating the amount of time it took to place an order I noticed that there were many outliers that lie well above the median. These outliers indicate that there may be UI/UX problems with the Jumpman23 app. By improving data acquisition on the types of items that customers are most likely to order we could provide data-driven restaurant recommendations to reduce latency to order placement and improve usabiliity of the app. 

```Python
#Create boxplot of time to order 
fig = plt.figure(figsize=(10,7))
ax = fig.add_subplot(111)

bp = ax.boxplot(order_length_df["how_long_it_took_to_order"], notch=True, vert=False)
ax.set_xlabel("Time to Place Order (minutes)")
ax.set_ylabel("Order Time")
plt.title("Latency to Order Placement")
```

![Latency to order placement](/Jumpman23/images/latency_to_order.png)

**Prep Time, Transit Time, and Total Order Time** 

```Python
new_df.describe()

delivery_id	customer_id	jumpman_id	item_quantity	how_long_it_took_to_order	pickup_lat	pickup_lon	dropoff_lat	dropoff_lon	prep_time	transit_time	total_order_time
count	5.214000e+03	5214.000000	5214.000000	3984.000000	2579.000000	5214.000000	5214.000000	5214.000000	5214.000000	4719.000000	4719.000000	5214.000000
mean	1.379183e+06	176061.059647	102783.959340	1.245231	7.698968	40.741633	-73.986928	40.744462	-73.985744	18.294248	14.041480	45.216308
std	6.469072e+04	116624.801790	48532.292063	0.781632	5.712095	0.022772	0.015002	0.025223	0.018061	11.715884	9.314023	19.687987
min	1.271706e+06	242.000000	3296.000000	1.000000	1.383292	40.665611	-74.015837	40.649356	-74.017679	0.001877	0.839419	3.047181
25%	1.322045e+06	77518.250000	61204.500000	1.000000	4.359893	40.724340	-73.996630	40.725440	-74.000193	10.369364	7.938390	32.122191
50%	1.375663e+06	129880.500000	113563.000000	1.000000	6.151847	40.735783	-73.988682	40.741115	-73.989367	15.333771	11.658124	42.020648
75%	1.435623e+06	294478.000000	143392.000000	1.000000	9.034514	40.758939	-73.980739	40.764239	-73.974409	22.977090	17.304775	54.240649
max	1.491424e+06	405547.000000	181543.000000	16.000000	73.221102	40.818082	-73.920980	40.848324	-73.924124	267.954044	119.190060	340.308810
```

```Python
#Create boxplot of prep time, transit time, and total time
fig = plt.figure(figsize=(10,7))
ax = fig.add_subplot(111)

bp = ax.boxplot([prep_time_df["prep_time"], transit_time_df["transit_time"], total_order_df["total_order_time"]], notch=True, vert=False)
ax.set_yticklabels(["Prep Time", "Transit Time", "Total Order Time"])
ax.set_xlabel("Time (minutes)")
plt.title("Prep Time, Transit Time, and Total Order Time in Minutes")   
```


![transit](/Jumpman23/images/preptransitorder_time.png)

Another area for improvement is in transit time. Average order time is around 45 minutes with a greater portion of this time spent on prep_time. In order to reduce the amount of time spent prepping restaurants with high order volume can prepare popular menu items ahead of time. Also, improving the system that assigns Jumpmen to orders based on pickup and dropoff location could improve total order time as well. Finally, ensuring that the Jumpman23 app is able to place orders ahead of time 100% of the time could help cut time on edge cases where the Jumpmen must make the order after arriving at the pickup location.  

**Delivery Vehicles** 

```Python
vehicle_type = {
    "Type" : ["Bicycle", "Car", "Truck", "Scooter", "Walker", "Van", "Motorcycle"],
    "Count" : [new_df[new_df["vehicle_type"] == "bicycle"].shape[0], new_df[new_df["vehicle_type"] == "car"].shape[0], new_df[new_df["vehicle_type"] == "truck"].shape[0], new_df[new_df["vehicle_type"] == "scooter"].shape[0], new_df[new_df["vehicle_type"] == "walker"].shape[0], new_df[new_df["vehicle_type"] == "van"].shape[0], new_df[new_df["vehicle_type"] == "motorcycle"].shape[0]]
}
fig = plt.figure(figsize=(10,7))
ax = fig.add_subplot(111)

bar = ax.bar(x=vehicle_type["Type"], height=vehicle_type["Count"])
ax.set_ylabel("Count of Vehicle Type")
plt.title("Delivery Vehicle Types")
```

![vehicle type](/Jumpman23/images/delivery_vehicle_types.png)

```Python
#Retrieve distance traveled by each vehicle type 
bicycle = new_df.loc[new_df["vehicle_type"] == "bicycle"]
bicycle_distance = bicycle["distance_miles"].tolist()
car = new_df.loc[new_df["vehicle_type"] == "car"]
car_distance = car["distance_miles"].tolist()
truck = new_df.loc[new_df["vehicle_type"] == 'truck']
truck_distance = truck["distance_miles"].tolist()
scooter = new_df.loc[new_df["vehicle_type"] == 'scooter']
scooter_distance = scooter["distance_miles"].tolist()
walker = new_df.loc[new_df['vehicle_type'] == "walker"]
walker_distance = walker["distance_miles"].tolist()
van = new_df.loc[new_df["vehicle_type"] == 'van']
van_distance = van["distance_miles"].tolist()
motorcycle = new_df.loc[new_df["vehicle_type"]=='motorcycle']
motorcycle_distance = motorcycle["distance_miles"].tolist()
data = [bicycle_distance, car_distance, truck_distance, scooter_distance, walker_distance, van_distance, motorcycle_distance]

#boxplot of distance travelled by vehicle type 
fig = plt.figure(figsize=(10,7))
ax = fig.add_subplot(111)

bp = ax.boxplot(data)
ax.set_ylabel("Distance Travelled (miles)")
ax.set_xticklabels(["bicycle", "car", "truck", "scooter", "walker", "van", "motorcycle"])
plt.title("Distance Travelled by Vehicle Type")
```

![delivery_distance](/Jumpman23/images/dist_traveled_by_tye.png)

Due to the nature of the NYC market it appears that the most popular mode of transportation for Jumpmen is by bicycle. Pairing the Jumpman23 app with directional apps tailored to bike riders could improve the transit time for the share of Jumpmen using bicycles for transportation. 

**Merchant Metrics**

```Python
#Find top ten pickup restaurants 
top_ten = pd.DataFrame(new_df["pickup_place"].value_counts().head(10))
top_ten_names = top_ten.index.tolist()

fig = plt.figure(figsize=(10,7))
ax = fig.add_subplot(111)

bar = ax.bar(x=top_ten_names, height=top_ten["pickup_place"])
ax.set_ylabel("Order Count")
plt.title("Top 10 Restaurants by Order Count")
plt.xticks(rotation=90)
```

![top ten](/Jumpman23/images/tp_ten_rest.png)

```Python
fig = plt.figure(figsize=(10,7))
ax = fig.add_subplot(111)

brp = ax.bar(x=avg_dropoff_latency_dict["Name"], height=avg_dropoff_latency_dict["time"])
plt.xticks(rotation=90)
plt.ylabel("Fulfillment Time (minutes)")
plt.title("Fulfillment Time for Top Ten Restaurants")
```

![top_fulfill](/Jumpman23/images/tp_ten_fulfillment.png)

```Python
fig = plt.figure(figsize=(10,7))
plt.plot(x, hhcounts)
plt.xticks(range(24))
plt.title("Order Volume by Hour")
plt.ylabel("Count of Orders")
```

![hourly_total](/Jumpman23/images/hourly_total.png)

According to the analysis, Shake Shack has the highest order volume of all restaurants and a fulfillment time that sits around the average for all merchants. Additionally, there are spikes in hourly order counts at 11 am and a greater spike at 6pm. Before these rush hours restaurants can prepare their most heavily requested items ahead of time in order to reduce latency to order fulfillment even during peak hours. Harboring a reputation as a delivery app that delivers on time even during peak hours can help beat out any competition that Jumpman23 may have in this market. 

**Pickup and Dropoff Heatmaps** 

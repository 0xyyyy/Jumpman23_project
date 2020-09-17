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



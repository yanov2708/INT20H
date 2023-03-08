# INT20H    team - shelby gang 😎

## Content:
* Main task
* Implementation
  - Data analysis 
  - Feature engineering
  - About building a grid on the map
  - About model
* What did we fail to do?
* [Our presentation](https://docs.google.com/presentation/d/1U1h3EjkSa-0i_aFUByYnwHISR9yoCbjD/edit?usp=share_link&ouid=112079393568654521016&rtpof=true&sd=true)
* [Test task](https://github.com/yanov2708/INT20H/blob/main/test_task_done_v2.ipynb)

## Task case by Uklon:

### Introduction
Estimated Time of Arrival (ETA) calculation.
is one of the most important ride-hailing services
platforms
 
The user opens the application, orders the desired one
car class and, after confirmation of the order, sees
ETA is the estimated time it will arrive by
destination

### Goal

It is necessary to predict the time for which
the user will arrive at the destination.
Using historical data for this
trips with actual end times
trips.

--- 
### Data description

The **orders.csv** file contains data:

`Id` - unique order identifier (order, reated by the passenger).

`running_time` - date and time when the trip started.
 
`completed_time` - date and time when completed
trip.

`route_distance_km` - distance in kilometers from the start
the route to the destination.

`delta_time` - is the actual time it took to complete
trip.

The **nodes.csv** file contains data:

`Id` – unique identifier of the order (the order created passenger).

`node_start` – initial node.

`node_finish` – final node.

`distance` – distance (gap) between nodes in meters.

`speed` – the average speed of other cars on it interval until the order is created.

* Node is one of the main elements of the OpenStreetMap data model. It consists of a single point in
space defined by its latitude, longitude and node id (node identifier)

* For one order from orders.csv, there will be several entries in nodes.csv that characterize the trip route,
divided into smaller straight intervals. They, in turn, are characterized by location (at the beginning and
at the end of the gap), the distance and the average speed at which the other drivers until the moment of placement
orders passed through this gap.

## Implementation

### Third party tools:

1. To read the Open Street Map file with the .osm extension, we used the `osmium` library, eventually getting the longitude and latitude of each node
2. To visualize our nodes on the map of Odessa, we used the `folium` library

### Data analysis 

The first thing you should pay attention to is that the trip data is given for one day - January 24, 2022

Next, looking at the distributions of `route_distance` and `delta_time`, we concluded that the data is clean, with the exception of literally a couple of rows that we removed.
We also mention that there were **no null values** in the dataset.

Deleted rows:

![image](https://user-images.githubusercontent.com/98317081/223690222-865e5ef7-b89b-49a0-9df9-dae49a392809.png)

Distribution charts

![image](https://user-images.githubusercontent.com/98317081/223688166-a2e34b57-08d9-416c-ba47-7d875f2387a5.png)

![image](https://user-images.githubusercontent.com/98317081/223688200-12c1bbba-d99b-46a3-a2db-29bfda490cde.png)

### Feature engineering

`minutes` - all our data is trips in 1 day, so it made sense to create one column with the time in minutes that has passed since the beginning of the day.

`total nodes on the route` - to calculate road congestion, as one of the components, we calculated the number of times how many connections of two nodes met in the routes.

`frequency, map with nodes` - putting all the nodes on a map divided into cells, we were able to easily see the most active and most saturated parts of the city. From here we got the frequency of trips from one cell to another.

`speed` and `route distance` - the basic features that entered the model almost without changes: the average speed between route nodes and the distance between boarding and disembarking

**About building a grid on the map:**

At first, we simply plotted the node points of the trip on the map, and then framed it with a grid, the size of the squares was chosen more intuitively than according to some kind of rule

![image](https://user-images.githubusercontent.com/98317081/223697583-317aef58-21a6-41bf-8b0b-1c1206b55e92.png)

A label was placed for each pair of initial and final cells, after which the number of such unique labels was counted. The larger the number -- the more popular the route, and therefore it can be with heavier traffic.

### About model

> Metrics - RMSE

For the base prediction, we just took the **average of the target** and got a value of 214 seconds

Next, we tried using **linear regression** and **Ridge** regression (L2 regularization) the result was already = 180 seconds and we wanted to compare it with the result of the tree-based **XGBRegressor**, and as it turned out, a learning rate = 0.05 and a maximum tree depth = 4 gave us the best result predictions = **121 seconds** without overfitting

\* Somewhere there should be a validation curv 

### What did we fail to do?

1. We tried to calculate the load of node combinations for the last two hours for each trip, but this did not improve the model, however, the feature increased the calculation time.
2. Weather: we easily found historical weather data for Odesa, but neither pandas nor bs4 helped us save the data to a dataframe, so after several hours of trying, we abandoned the idea.
3. It was also interesting to see if the altitude of the starting and ending points of the route affected the duration of the trip, but we were unable to conveniently obtain the data.
4. It was also possible to get more information from the Open Street Map

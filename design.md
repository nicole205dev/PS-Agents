**#1. Designing the Agent**
**##1.1 The problem**
A trailer has to carry goods from Nairobi to Kisumu using Kenya's highways. The route passes through the Nairobi region, the Central Highlands, the Rift Valley and the Lake Victoria Basin. Our agent is the system that decides which towns the trailer should pass through, and then drives it there. We want the trip to be as short as possible.
**##1.2 PEAS**

**Performance measure**
The main measure is the total distance travelled from Nairobi to Kisumu, and the shorter the better. We also care about travel time, fuel use, arriving on time, and keeping the cargo and the driver safe.
**Environment**
The highways and towns between Nairobi and Kisumu, other vehicles and pedestrians, weather, road condition and roadworks, steep sections like the Rift Valley escarpment, and weighbridges and police checks.
**Actuators**
Steering, accelerator, brakes, gearbox, indicators and lights, and a screen or voice output that tells the driver which way to go.
**Sensors**
	GPS, cameras and radar, speedometer and odometer, fuel gauge, engine and tyre sensors, and a live traffic or road-closure feed.
 ## 1.3 Environment properties

We looked at the environment at two levels. At the map level the agent is only planning the route. At the road level it is actually driving.

| Property | Map level (route planning) | Road level (driving) | Why |
|---|---|---|---|
| Observable | Fully observable | Partially observable | On the map the agent knows every town and distance. On the road it cannot see around bends or know what other drivers will do. |
| Number of agents | Single agent | Multi-agent | Planning a route does not depend on anyone else. Driving does, because of other vehicles, pedestrians and police. |
| Deterministic | Deterministic | Stochastic | In the model, "go from Gilgil to Nakuru" always ends in Nakuru. In real life, traffic, weather and breakdowns can change the outcome. |
| Episodic or sequential | Sequential | Sequential | Each town we choose affects the choices that come after it and the final cost. |
| Static or dynamic | Static | Dynamic | The map does not change while we plan. Traffic and weather change while the trailer moves. |
| Discrete or continuous | Discrete | Continuous | There is a limited set of towns and roads. Speed, steering angle and position can take any value. |
| Known or unknown | Known | Mostly known | The agent knows which towns connect and how far apart they are, but not every local road condition. |
# 2. The Graph and the Search Tree

## 2.1 Abstraction

A real trip has far too much detail to plan with. There are thousands of bends, junctions, lanes and speed changes. To keep the problem manageable we used abstraction, which just means keeping the details that matter for choosing a route and dropping the rest.

| Real-world detail | What we did with it |
|---|---|
| Exact GPS position, lane and speed | Dropped it. The state is simply the town the trailer is in. |
| Steering, braking and gear changes | Dropped it. One action is "drive along the highway to the next town". |
| Traffic, weather, road quality and hills | Dropped it. We assumed they are the same on every road. |
| All the bends and junctions between two towns | Merged into one edge with one cost, the driving distance in km. |
| Small towns off the main highways | Left out. We only kept the major towns on the main routes. |

We used three levels of abstraction:

1. **The real world:** continuous positions and driving actions.
2. **The town graph:** towns as nodes and highways as edges with distances. We search at this level.
3. **The regions:** the Nairobi region, Central Highlands, Rift Valley and Lake Victoria Basin as four big nodes. This gives the overall picture of the journey: from the Nairobi region, through the Rift Valley, to the Lake Victoria Basin.

   

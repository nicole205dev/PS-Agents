# PS-Agents: Finding the Shortest Route for a Trailer from Nairobi to Kisumu (Updated with Verified Distances)

**Group members:** Salim Hatibu (190000), Peter Atonga (169614), Nicole Momanyi (168320)

---

## 1. Designing the Agent

### 1.1 The problem
A trailer has to carry goods from Nairobi to Kisumu using Kenya's highways. Along the way it passes through the Nairobi region, the Central Highlands, the Rift Valley, and the Lake Victoria Basin. The agent is the system that decides which towns the trailer goes through and then drives it there. We want the trip to be as short as possible.

### 1.2 PEAS

| Part | In our trailer problem |
| :--- | :--- |
| **Performance measure** | The main thing is the total distance from Nairobi to Kisumu, so shorter is better. Travel time, fuel, safety of the driver and cargo, and arriving on time also count. |
| **Environment** | The highways and towns between Nairobi and Kisumu. Other vehicles and pedestrians, weather, potholes and roadworks, steep parts like the escarpment, weighbridges, and police checks. |
| **Actuators** | Steering, accelerator, brakes, gearbox, indicators and lights, and a screen or voice prompt telling the driver where to go. |
| **Sensors** | GPS, cameras and radar, speedometer, odometer, fuel gauge, engine and tyre sensors, and a live feed of traffic and road closures. |

We treat this as a goal-based agent. It knows its goal which is Kisumu, it has a map of the roads, and it searches for the moves that reach the goal at the lowest cost.

### 1.3 Environment properties

We looked at the environment in two ways. On the map the agent is only planning a route. On the road it is actually driving.

| Property | Map level (planning) | Road level (driving) | Reason |
| :--- | :--- | :--- | :--- |
| **Observable** | Fully observable | Partially observable | The map shows every town and distance. On the road the agent cannot see around a bend or know what other drivers will do. |
| **Agents** | Single agent | Multi-agent | A route plan does not depend on anyone else. Driving does, because of other vehicles, pedestrians, and traffic police. |
| **Deterministic** | Deterministic | Stochastic | In our model, going from Gilgil to Nakuru always puts us in Nakuru. In real life, traffic, weather, or a breakdown can change that. |
| **Episodic or sequential** | Sequential | Sequential | Each town we pick affects the next choice and the final cost. |
| **Static or dynamic** | Static | Dynamic | The map does not change while we plan, but traffic and weather change while the trailer moves. |
| **Discrete or continuous** | Discrete | Continuous | The towns and roads are a fixed, countable set. Speed, steering angle, and position can take any value. |
| **Known or unknown** | Known | Mostly known | The agent knows which towns connect and how far apart they are, but not every local road condition. |

For the search part of this exercise, the environment is fully observable, single-agent, deterministic, sequential, static, discrete, and known. Graph search works well for that.

---

## 2. The Graph and the Search Tree

### 2.1 Abstraction

A real journey has too much detail to plan with: thousands of bends, junctions, lanes, and speed changes. So we used abstraction, meaning we kept only what matters for choosing a route and dropped the rest.

| Real-world detail | What we did with it |
| :--- | :--- |
| **Exact GPS position, lane, and speed** | Dropped. The state is just the town the trailer is in. |
| **Steering, braking, and gear changes** | Dropped. One action is "drive along the highway to the next town". |
| **Traffic, weather, road quality, and hills** | Dropped. We assumed they are the same on every road. |
| **All the bends and junctions between two towns** | Combined into one edge with one cost, the driving distance in km. |
| **Small towns off the main highways** | Left out. We only kept the major towns on the main routes. |

We used three levels of abstraction:

1. **The real world:** continuous positions and driving actions.
2. **The town graph:** towns as nodes and highways as edges with distances. This is where we do the search.
3. **The regions:** the Nairobi region, Central Highlands, Rift Valley, and Lake Victoria Basin as four big nodes. This shows the overall trip, from the Nairobi region through the Rift Valley to the Lake Victoria Basin.

```mermaid
flowchart LR
    NBI["Nairobi Region<br/>Nairobi, Limuru, Thika"] --> CH["Central Highlands<br/>Murang'a, Nyeri, Nanyuki, Embu, Nyahururu"]
    NBI --> RV["Rift Valley<br/>Naivasha, Gilgil, Nakuru, Narok, Londiani, Kericho, Bomet, Eldoret"]
    CH --> RV
    RV --> LVB["Lake Victoria Basin<br/>Kisumu, Muhoroni, Kisii, Kakamega, Bungoma, Busia"]

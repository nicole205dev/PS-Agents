#PS-Agents: Finding the Shortest Route for a Trailer from Nairobi to Kisumu (Updated with Verified Distances)
Group members: Salim Hatibu (190000), Peter Atonga (169614), Nicole Momanyi (168320)

##1. Designing the Agent
###1.1 The problem
A trailer has to carry goods from Nairobi to Kisumu using Kenya's highways. Along the way it passes through the Nairobi region, the Central Highlands, the Rift Valley, and the Lake Victoria Basin. The agent is the system that decides which towns the trailer goes through and then drives it there. We want the trip to be as short as possible.

###1.2 PEAS
Part	In our trailer problem
Performance measure	The main thing is the total distance from Nairobi to Kisumu, so shorter is better. Travel time, fuel, safety of the driver and cargo, and arriving on time also count.
Environment	The highways and towns between Nairobi and Kisumu. Other vehicles and pedestrians, weather, potholes and roadworks, steep parts like the escarpment, weighbridges, and police checks.
Actuators	Steering, accelerator, brakes, gearbox, indicators and lights, and a screen or voice prompt telling the driver where to go.
Sensors	GPS, cameras and radar, speedometer, odometer, fuel gauge, engine and tyre sensors, and a live feed of traffic and road closures.
We treat this as a goal-based agent. It knows its goal which is Kisumu, it has a map of the roads, and it searches for the moves that reach the goal at the lowest cost.

###1.3 Environment properties
We looked at the environment in two ways. On the map the agent is only planning a route. On the road it is actually driving.

Property	Map level (planning)	Road level (driving)	Reason
Observable	Fully observable	Partially observable	The map shows every town and distance. On the road the agent cannot see around a bend or know what other drivers will do.
Agents	Single agent	Multi-agent	A route plan does not depend on anyone else. Driving does, because of other vehicles, pedestrians, and traffic police.
Deterministic	Deterministic	Stochastic	In our model, going from Gilgil to Nakuru always puts us in Nakuru. In real life, traffic, weather, or a breakdown can change that.
Episodic or sequential	Sequential	Sequential	Each town we pick affects the next choice and the final cost.
Static or dynamic	Static	Dynamic	The map does not change while we plan, but traffic and weather change while the trailer moves.
Discrete or continuous	Discrete	Continuous	The towns and roads are a fixed, countable set. Speed, steering angle, and position can take any value.
Known or unknown	Known	Mostly known	The agent knows which towns connect and how far apart they are, but not every local road condition.
For the search part of this exercise, the environment is fully observable, single-agent, deterministic, sequential, static, discrete, and known. Graph search works well for that.

##2. The Graph and the Search Tree
###2.1 Abstraction
A real journey has too much detail to plan with: thousands of bends, junctions, lanes, and speed changes. So we used abstraction, meaning we kept only what matters for choosing a route and dropped the rest.

Real-world detail	What we did with it
Exact GPS position, lane, and speed	Dropped. The state is just the town the trailer is in.
Steering, braking, and gear changes	Dropped. One action is "drive along the highway to the next town".
Traffic, weather, road quality, and hills	Dropped. We assumed they are the same on every road.
All the bends and junctions between two towns	Combined into one edge with one cost, the driving distance in km.
Small towns off the main highways	Left out. We only kept the major towns on the main routes.
We used three levels of abstraction:

The real world: continuous positions and driving actions.

The town graph: towns as nodes and highways as edges with distances. This is where we do the search.

The regions: the Nairobi region, Central Highlands, Rift Valley, and Lake Victoria Basin as four big nodes. This shows the overall trip, from the Nairobi region through the Rift Valley to the Lake Victoria Basin.

Code snippet
flowchart LR
    NBI["Nairobi Region<br/>Nairobi, Limuru, Thika"] --> CH["Central Highlands<br/>Murang'a, Nyeri, Nanyuki, Embu, Nyahururu"]
    NBI --> RV["Rift Valley<br/>Naivasha, Gilgil, Nakuru, Narok, Londiani, Kericho, Bomet, Eldoret"]
    CH --> RV
    RV --> LVB["Lake Victoria Basin<br/>Kisumu, Muhoroni, Kisii, Kakamega, Bungoma, Busia"]
###2.2 Problem formulation
States: the towns on the graph

Initial state: Nairobi

Goal test: is the trailer in Kisumu?

Actions: drive from a town to a neighbouring town

Result of an action: the trailer is now in that neighbouring town

Path cost: the total of the road distances in km

###2.3 Directed graph
Highways can be driven both ways, so each road is drawn as two arrows, one in each direction, with the same distance. The numbers on the edges are updated, accurate distances in km.

Code snippet
flowchart LR
    Nairobi <-->|37.5| Limuru
    Nairobi <-->|46.1| Thika
    Nairobi <-->|142.0| Narok
    Limuru <-->|56.3| Naivasha
    Thika <-->|46.6| Murang'a
    Thika <-->|91.2| Embu
    Murang'a <-->|62.2| Nyeri
    Nyeri <-->|61.0| Nanyuki
    Nyeri <-->|104.0| Nyahururu
    Naivasha <-->|30.4| Gilgil
    Gilgil <-->|41.1| Nakuru
    Nakuru <-->|66.2| Nyahururu
    Nakuru <-->|64.9| Londiani
    Londiani <-->|49.0| Kericho
    Londiani <-->|116.0| Eldoret
    Kericho <-->|39.6| Muhoroni
    Kericho <-->|75.2| Bomet
    Muhoroni <-->|60.7| Kisumu
    Narok <-->|81.0| Bomet
    Bomet <-->|82.4| Kisii
    Kisii <-->|107.0| Kisumu
    Eldoret <-->|83.9| Kakamega
    Kakamega <-->|50.9| Kisumu
    Kakamega <-->|60.5| Bungoma
    Bungoma <-->|59.2| Busia
    Busia <-->|113.0| Kisumu
###2.4 Distances (Updated Accurate Values)
From	To	Distance (km)
Nairobi	Limuru	37.5
Nairobi	Thika	46.1
Nairobi	Narok	142.0
Limuru	Naivasha	56.3
Thika	Murang'a	46.6
Thika	Embu	91.2
Murang'a	Nyeri	62.2
Nyeri	Nanyuki	61.0
Nyeri	Nyahururu	104.0
Naivasha	Gilgil	30.4
Gilgil	Nakuru	41.1
Nakuru	Nyahururu	66.2
Nakuru	Londiani	64.9
Londiani	Kericho	49.0
Londiani	Eldoret	116.0
Kericho	Muhoroni	39.6
Kericho	Bomet	75.2
Muhoroni	Kisumu	60.7
Narok	Bomet	81.0
Bomet	Kisii	82.4
Kisii	Kisumu	107.0
Eldoret	Kakamega	83.9
Kakamega	Kisumu	50.9
Kakamega	Bungoma	60.5
Bungoma	Busia	59.2
Busia	Kisumu	113.0
###2.5 Search tree
The root of the tree is Nairobi. Under each town we listed the towns the trailer can go to next, without allowing loops. The number next to each town is 
𝑔
(
𝑛
)
g(n), the cumulative distance traveled from Nairobi. Nodes marked GOAL are where the trailer reaches Kisumu.

Code snippet
flowchart TD
    A["Nairobi<br/>g=0"] --> B["Limuru<br/>g=37.5"]
    A --> C["Thika<br/>g=46.1"]
    A --> D["Narok<br/>g=142.0"]

    B --> E["Naivasha<br/>g=93.8"]
    E --> F["Gilgil<br/>g=124.2"]
    F --> G["Nakuru<br/>g=165.3"]
    G --> H["Londiani<br/>g=230.2"]
    G --> I["Nyahururu<br/>g=231.5"]
    I --> I1["Nyeri<br/>g=335.5"]

    H --> J["Kericho<br/>g=279.2"]
    H --> K["Eldoret<br/>g=346.2"]
    J --> L["Muhoroni<br/>g=318.8"]
    J --> M["Bomet<br/>g=354.4"]
    L --> N(["KISUMU<br/>g=379.5 GOAL"])
    M --> M1["Kisii<br/>g=436.8"]
    K --> O["Kakamega<br/>g=430.1"]
    O --> P(["KISUMU<br/>g=481.0 GOAL"])
    O --> P1["Bungoma<br/>g=490.6"]

    C --> Q["Murang'a<br/>g=92.7"]
    C --> R["Embu<br/>g=137.3"]
    Q --> S["Nyeri<br/>g=154.9"]
    S --> T["Nanyuki<br/>g=215.9"]
    S --> U["Nyahururu<br/>g=258.9"]
    U --> V["Nakuru<br/>g=325.1"]

    D --> W["Bomet<br/>g=223.0"]
    W --> X["Kisii<br/>g=305.4"]
    W --> Y["Kericho<br/>g=298.2"]
    X --> Z(["KISUMU<br/>g=412.4 GOAL"])
    Y --> Y1["Muhoroni<br/>g=337.8"]
    Y1 --> Y2(["KISUMU<br/>g=398.5 GOAL"])

    style N fill:#b7e4c7,stroke:#2d6a4f,stroke-width:3px
The tree yields four primary routes to Kisumu with the updated distances:

Route	Total Distance (km)
Nairobi → Limuru → Naivasha → Gilgil → Nakuru → Londiani → Kericho → Muhoroni → Kisumu	379.5
Nairobi → Narok → Bomet → Kericho → Muhoroni → Kisumu	398.5
Nairobi → Narok → Bomet → Kisii → Kisumu	412.4
Nairobi → Limuru → Naivasha → Gilgil → Nakuru → Londiani → Eldoret → Kakamega → Kisumu	481.0
##3. Finding the Best Route
###3.1 Method
Using uniform-cost search (Dijkstra's algorithm), we expand nodes based on the lowest cumulative path cost 
𝑔
(
𝑛
)
g(n) from Nairobi. The search terminates when Kisumu is popped from the priority queue as the cheapest destination.

###3.2 Working
Step	Town expanded	Distance so far (g)	What we found / Updated Frontier
1	Nairobi	0	Limuru 37.5, Thika 46.1, Narok 142.0
2	Limuru	37.5	Naivasha 93.8
3	Thika	46.1	Murang'a 92.7, Embu 137.3
4	Murang'a	92.7	Nyeri 154.9
5	Naivasha	93.8	Gilgil 124.2
6	Gilgil	124.2	Nakuru 165.3
7	Narok	142.0	Bomet 223.0
8	Nyeri	154.9	Nanyuki 215.9, Nyahururu 258.9
9	Nakuru	165.3	Londiani 230.2, Nyahururu 231.5 (keeps 231.5 via Nakuru or 258.9 via Nyeri)
10	Nanyuki	215.9	Dead end
11	Bomet	223.0	Kisii 305.4, Kericho 298.2
12	Londiani	230.2	Kericho 279.2, Eldoret 346.2
13	Embu	137.3	Dead end
14	Nyahururu	231.5	Nakuru 297.7 (already expanded)
15	Kericho	279.2	Muhoroni 318.8, Bomet 354.4 (already visited cheaper)
16	Kisii	305.4	Kisumu 412.4
17	Muhoroni	318.8	Kisumu 379.5
18	Kisumu	379.5	Goal reached, stop.
3.3 Result
The optimal path is:

Nairobi → Limuru → Naivasha → Gilgil → Nakuru → Londiani → Kericho → Muhoroni → Kisumu

Leg	Distance (km)	Running Total (km)
Nairobi to Limuru	37.5	37.5
Limuru to Naivasha	56.3	93.8
Naivasha to Gilgil	30.4	124.2
Gilgil to Nakuru	41.1	165.3
Nakuru to Londiani	64.9	230.2
Londiani to Kericho	49.0	279.2
Kericho to Muhoroni	39.6	318.8
Muhoroni to Kisumu	60.7	379.5
The optimal path cost is 379.5 km.

###3.4 Analysis
Uniform-cost search successfully identified the minimal-cost route using the updated accurate segment measurements.

The alternative southern route via Narok, Bomet, and Kericho totals 398.5 km (~19 km longer).

Bypassing the Rift Valley through the Central Highlands (Thika → Murang'a → Nyeri) incurs higher cumulative edge costs, making it suboptimal for trailer freight routing.

##4. Evidence of Group Work
###4.1 Meeting minutes
Date and time: 15/10/2026, 5:30 PM

Where: Student Centre, Strathmore University

Present: Nicole Momanyi, Peter Atonga, Salim Hatibu

What we discussed:

Re-verifying road segments and updating distance metrics with precise Google Maps data.

Adjusting the search tree weights and recalculating the priority queue steps for Uniform-Cost Search.

Finalizing documentation updates and repository structure.

Tasks and who did them:

Task	Member	Deadline
PEAS and environment properties review	Nicole	15/10/2026
Graph adjustments and mermaid tree updates	Salim	15/10/2026
Re-checking accurate distances on Google Maps	Peter	15/10/2026
Uniform-cost search recalculations	Peter / Salim	15/10/2026
Final review and formatting	Nicole	15/10/2026

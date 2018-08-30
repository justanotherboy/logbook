# OSPF Areas

OSPF areas are used for hierarchy and escalability. Areas will achieve topology and route summarization.

Scalability is a function of two variables
- How complex is the topology.
  - How many routers are in the Area
  - Large flooding domains means a lots of SPF runs
- How much reachability information is there
  - How many routes are being advertised
  - Large routing tables means it takes longer to flood

Topology summarization is achieved through OSPF areas
- Hide the topology details of other areas (Distance Vector protocols always hide this information)
- Areas don't hide reachability information, though. Prefix summarization reduce the number of routes
- An area defines a flooding domain
    - All devices in the area agree on the topology
    - Changes inside the area require LSA flooding and full SPF
    - Most times incremental SPF won't help with modern CPUs.
- Routing between areas hides topology details
    - Inter-area routing is similar to distance vector
    - Changes outside the area don't always require LSA flooding or SPF
    - Limits impact on router resources

## Router types

- Backbone routers
    - Have at least on link in area 0
- Internal routers
    - All links in one non-backbone area
- Area Border Router (ABR)
    - Links in both area 0 and in non-backbone area(s)
    - Used to summarize information between area 0 and non-backbone area and viceversa
- Autonomous System Boundary Router (ASBR)
    - At least one link in the OSPF domain
    - At least one link outside the OSPF domain
    - Used to redistribute information to/from other routing domains and OSPF

## Area types

### Backbone area

- Area 0
- Used to summarize topology information between other areas
- Traffic from one area to another must pass through area 0
- Must be contiguos (Mechanism to avoid loops)

### Non-backbone areas

- Not backbone areas
- Must use connections to area 0 to reach other areas


### Stub areas

- A good candidate for a stub area is an area which have a single exit point.
- Filtering is enforced at a common transit point of the OSPF topology (the ABR)
- ABR controls which LSAs enter the area
    - Type-3, Type-4 and/or Type-5 are filtered depending on stub type
- Reachability information removed is then replaced with a default route
- All routers must agree on the stub flag
    - Part of adjacency negotiation.

Type of stub areas:
- Stub area
  - Stop external routes
- Totally Stubby Area
  - Stops inter-area and external routes
- Not-So-Stubby Area (NSS)
  - Stops external routes, but allows local redistribution
- Not-So-Totally-Stubby Area
  - Stops inter-area and external routes, but allows local redistribution

#### Stub area

- Use ABR to reach external routes through the ASBR
- ABR removes LSAs 4 (ASBR) & 5 (External)
- ABR originates a default route

#### Totally Stuby Area

- Use ABR to reach external and inter-area routes
- ABR removes LSA 3 (Inter-Area), 4 (ASBR), and 5 (External)
- ABR originates a default route
- Cisco "propietary" can be achieved with stub areas and summarization

#### Not-So-Stubby-Area

- Filter like a stub area, but make an exception external routes into the area
- Redistributing router (internal ASBR) generates LSA 7 (NSSA External)
- ABR changes NSSA External (Type 7) into External (Type 5) into area 0
- ABR removes LSAs 4 (ASBR) and 5 (External)
- ABR does not automatically originate default route

#### Not-So-Totally Stubby Area

- Filter like a totally stubby area, but make an exception external routes into the area
- Redistributing router generates NSSA External (LSA Type 7)
- ABR changes NSSA External (Type 7) into External (Type 5) into area 0
- ABR removes LSAs 3 (Inter-Area), 4 (ASBR) and 5 (External)
- ABR originate default route
- Cisco "propietary" can be achieved with stub areas and summarization


# OSPF network types

- OSPF behavior changes based on the type media is configured
- OSPF defines different "network types" to deal with different characteristics
    - Timers
    - Unicast vs. Multicast
    - Adjacency relations
    - How is the next-hop calculated
- Network type has to be compatible to form an adjacency. The use (or lack of use) of LSA 2 is the one that makes them compatible.
    - If the network types don't match the adjacency will be established, but the routing bit for the LSA will be disabled because SPF will fail.
- Even if OPSF is running on a single interface might generate more than one link. One to indicate the connection to another router (to run SPT) and one (or more) for the actual routes in the tree.

Type of OSPF netwoks:
1. Broadcast
2. Non-Broadcast
3. Point-to-Point
4. Point-to-Multipoint
5. Point-to-Multipoint Non-Broadcast
6. Loopback
7. Virtual Links

## Loopback

- Software/Hardware loopbacks
- Advertises link as /32 stub host router

## Point-to-Point medias

### Point-to-Point

- HDCL, PPP, GRE
- Hellos sent to multicast 224.0.0.5 (All SPF Routers)
- No DR/BDR
- Support only two neighbors on the link

### Point-to-Multipoint

- Hellos sent to multicast 244.0.0.5 (All SPF Routers)
- No DR/BDR
- Specail next-hop processing
- Usually the best design option for partial mesh NBMA networks

### Point-to-Multipoint Non-Broadcast

- Hellos are sent as unicast
- No DR/BDR
- Allows per VC OSPF cost over NBMA
- Special next-hop processing
- Cisco propietary

## Multi-Access medias

### Designated Router

- Designated Router (DR) and the Backup Designated Router (BDR) are elected in Multi-Access medias

#### Designated Router

- Forms adjacency with all routers on the link
- Listens for LSUs (224.0.0.6)
- Re-floods LSUs back to the segment (224.0.0.5)
- Does not modify the next-hop value

#### Backup Designated Router (BDR)

- Used for redundancy of DR
- Does not re-flood LSUs

#### DROthers

- All other routers on the link
- Form FULL adjacency with DR & BDR
- 2-way adjacency with each other

### DR/BDR Election

- Neighbors attached to the network and having established bidirectional communication with Router X is examined. Discard all routers with priority of 0
1. Get current values of network's DR and BDR for later comparison
2. Elect the new BDR as follow:
    1. It should have not be declared DR
    2. If there are routers declared BDRs in the hello packets the one with highest priority is declared BDR, in case of a tie, the highest router-id is chosen
    3. If there are not declared BDRs routers in the hello packet the one with highest priority is declared BDR, in case of a tie, the highest router-id is chosen
3. Elect the new DR as follow:
    1. If there are routers declared DRs in the hello packets the one with highest priority is declared DR, in case of a tie, the highest router-id is chosen
    2. If there are not declared DRs routers in the hello packet the elected BDR is promoted to DR
4. When a router is newly DR or BDR, or is no longer DR or BDR, repeat steps 2 and 3 and then proceed to step 5. To avoid the same router to be DR or BDR.
5. Router's interface state should be set accordingly.
6. For the DR it should start sending Hello packets to the neighbors (multicast or unicast as required by the network type)
7. Check broken adjacencies with AdjOK on all neighbors that are at least in 2-way state

- If there is not already a DR/BDR the router will wait for the WAIT timer to expire before electing DR/BDR. In OSPF there is no pre-emption of DR/BDR, so most of the time the DR will be the router that was first to the segment
- A way to force pre-emption (not recommended) is to filter inbound OSPF packets and this will make a router to declare itself DR (after the Wait interval) and then remove the filter to have two DRs in the segment and this will restart the negotation
- A spoke can preempt the hub in a DMVPN network if there is no communication while the spoke is in wait state and it has higher priority than the hub (or equal priority and higher router-id)
 
### Multi-Access Broadcast

- Ethernet, Token Ring, FDDI
- Hellos are sent as multicast 244.0.0.5 (All SPF Routers) 224.0.0.6 (All DR Routers)
- Use DR & BDR
- The routers needs to be able to flood to the DR and the DR to all the routers
- A small improvement is to change the network type of broadcast network to point to point when only two neighbors are used

### Multi-Access non-broadcast

- Frame Relay, ATM
- Hellos are sent as unicast
- Uses DR & BDR
- The routers needs to be able to flood to the DR and the DR to all the routers.

## Virtual Links

A virtual link will connect an area not connected to the backbone area through a transit area. It is also possible to use a virtual link to connect a discontiguous backbone area.

Can be used for:
- Discontiguous Areas (not connection to area 0)
    - Broken area design can be fixed adding new area 0 links and adjacencies.
    - Links can be physical or virtual.
    - Virtual links are a form of virtual area 0 adjacencies.
    - GRE is software routed, not efecient for virtual links.
- Traffic Engineering
    - Path selection with virtual links (intra-area routing)
    - Route traffic between two areas without using area 0. This is a very corner case

- Cost is calculated using the area SPT (The cost of the link may exceed the maximum cost)
- Hellos are suppressed
- Entries learned using the virtual link do not age in the database
- Use an already built SPT between ABRs to heal the topology
- Multi-hop unicast adjacency between ABRs

### Virtual links caveats

- Endpoints must be reachable via a normal area
- Transit area must not have filtering
- Inherits cost from SPT cost between endpoints
- Runs as demand circuit
    - Errors in config could be hidden until flooding occurs
- Area running the virtual link has to have the transit capability


## Cisco commands

interface 
ip ospf network [point-to-point | point-to-multipoint | point-to-multipoint non-broadcast | broadcast | non-broadcast]

router process
neighbor [ip address] (for non-broadcast network types) 
area [transit-area] virtual-link [router-id] (to set up virtual link)


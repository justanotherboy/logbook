# OSPF Adjacencies

- Hello contains attributes the neighbors should agree on to make an adjacency.
- It might have an neighbor relationship but not adjacency.
- Exchange the Link-State Database once the Adjacency is made.
- Adjacency occurs when connected neighbors use hello packets to agree on unique and common attributes.
- Supported features will be exchanged to check what the neighbor is capable of
    - link-local signaling
    - area transit capability
    - Not-So-Stuby-Area
    - Single TOS routes
    - Incremental SPF

## Adjacency attributes

The Hello packet contains a series of attributes. To form an adjacency there are some unique attributes to the routers and common attributes the router should agree on.

### Hello Packet Attributes

- Local Router-ID
- Local Area-ID
- Local Interface Subnet Mask
- Local Interface Priority
- Hello Interval
- Dead Interval
- Authentication Type and Password
- DR/BDR Addresses
- Options
- Router IDs of other neighbors on the same link
- Other optional capabilities

### Unique Attributes

- Router ID
  - Manual configuration
  - Highest active loopback IP
  - Highest active interface IP

- Interface IP Address
  - OSPFv2 interface IP should be unique
  - OSPFv3 interface link-local IPv6 should be unique

### Common Attributes

- Interface Area-ID
- Hello Interval & Dead Interval
- Interface network address
- MTU
- Network Type
- Authentication
- Stub Flags

## Adjacency State Machine

1. Down/Attempt
    - Initial state when no information has been exchanged
    - Attempt is used in NBMA network
2. Init
    - A hello packet has been received from a neighbor
    - Not a 2-way conversation yet
3. 2-Way
    - Bidirectional conversation between routers
    - Local router IP address is in neighbor's hello packet
    - Minimum state to be considered in the Designated Router election
    - DROTHERs form this adjacency with other DROTHERs
4. ExStart
    - Master/Slave negotiation to select who is dictating the updates
5. Exchange
    - Routers exchange DataBase Descriptors (DBD) packets
    - DBD contains Link-State Advertisements (LSA) headers only
    - Master increments the sequence number of the DBDs once is ACK by the slave
    - Routers send LSAs requests and LSAs updates
    - Check if a more up to date LSA is available from the neighbor
6. Loading
    - Actual exchange of LSA updates
    - All LSAs are ACK
7. Full
    - Routers are fully adjacent with each other
    - All router and network LSAs are exchanged
    - Databases are fully synchronized
    - Normal state between neighbors except for DROTHERs <-> DROTHERs

## Adjacency complete

### SPF calculation 

- Once database are sync the path selection start using the SPF algorithm
- The cost attributes of the LSA is used to calculate the shortest path first to other nodes in the graph (not to prefixes)
- It is possible to have Equal Cost Multi-Path (ECMP)

### Maintenance of adjacency

- Hello packets are used to track neighbor changes
    - Hello 10 seconds, Dead 40 seconds for
        - Broadcast
        - Point to Point
    - Hello 30 seconds, Dead 120 seconds for
        - Non-Broadcast MultiAccess
        - Point to Multipoint
- The protocol can be tunned for subsecond detection. E.g. Bi-Directional Forwarding Detection.
- LSA fields used to track topology changes. When an LSA is received it will be checked aginst the database for changes:
    - New sequence number
    - Age
        - Flood occurs after 30 minutes (Do not need to run SPF)
        - Max Age is 60 minutes to withdraw the LSA (can be used to poison the LSA)
    - Checksum

## Troubleshooting adjacencies

### Transport problems

Problems with the physical or link layer usually is up to 2-Way

- Mismatch interface MTU but the attribute check is ignored (Stuck in Exstart)
- 

### Attribute problems

Problems with OSPF attributes

- If the Hello/Dead Intervals don't match the routers won't go into 2-way state.
- Mismatch MTU attribute
- Flood war: Adding and withdrawing attributes from an SLA. This usually happens with a duplicated router ID.

### Routers/Routes in OSPF Database but not in the Routing Table

- Network type mismatch
- Wrong address assignment in dual serial link
- One side of point-to-point link included in the wrong majornet or subnet
- One side is unnumbered and the other side is numbered
- Broken PVC in fully meshed Frame Relay
- Forwarding address known via external route
- Distribute list is blocking the route

### Cisco commands

show ip ospf neighbors
show ip ospf database
debug ip ospf adjacency
debug ip packet


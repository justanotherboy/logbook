# OSPF

## Overview

- Is an Interior Gateway Protocol
- Is Link-State based (all routers know the complete topology in their area)
- Uses Dijkstra Shortest Path Fist algorithm
- Supports CIDR and VLSM (OSPFv2)
- Most of the time uses multicast to send and receive packets
    - 224.0.0.5 All SPF routers
    - 224.0.0.6 Designated routers
- Uses areas for scalabilty
- It is an Open standard
- Actively looks for neighbors using hello packets. These packets are used to track neighbor adjacencies too
- Defined as IP protocol 89
- Metric is based on cost (it is arbitrary but usually is based on bandwidth)
- Supports authentication
- Supports event driven incremental updates
- OSPF won't route to IP prefixes, the algorithm solves the shortest path to other nodes (routers) and will route to the prefixes because is an attribute of the node

## Overview of operation

1. Discover OSPF Neighbors and Exchange Topology information
2. All routers agree on the topology
3. Choose best path using SPF
4. Maintain the Topology

## Cisco Comands

ip routing 

routing process:
router ospf [process-id]
auto-cost reference-bandwidth [bandwidth in Mb/s]
network [address] [wildcard] area [area-id]

interface configuration:
ip ospf [process-id] area [area-id] {secondaries none}

show ip ospf
show ip ospf interface [brief]
show ip ospf neighbor


# BGP peering

## iBGP

iBGP learned routes cannot be advertised to other iBGP neighbors in order to prevent loops 

This rules implies iBGP requires either
    - Fully meshed iBGP peerings
    - Route reflectors
    - Confederations

### iBGP full mesh

#### Advantages

- All BGP peers learn all possible egress paths
    - Path Diversity
- Optimal Traffic Flows
    - All BGP peers learn the closest egress path
    - Path selection by default would be based on IGP metric to egress router (Hot potato routing)

#### Disadvantage

- Control plane scaling
- Needs n(n-1)/2 peerings
- Operationally hard to scale
    - Adding or changing peering config is administratively prohitive

### Route Reflectors

- Hide the topology details to its clients,  make the routing decision locally and send only the best path to its "clients"
- The "clients" sends update to its RR
- The RR sends the udpate to its "clients"
- RR doesn't modify other attributes when reflecting to avoid loops
- RR discards routes received with its own Cluster-ID to avoid loops
- Clients uses Originator-ID for loop prevention
- RR serves only one address-family avoiding fate sharing of services
- It can have three different type of clients:
    - eBGP
        - Pass routes to all other clients
    - iBGP Client Peers
        - Pass routes to all other clients
    - iBGP Non-Client Peers
        - Pass routes to eBGP peers and Clients
- The way a RR learns a route depends on the order a RR learns a route and how advertise
    - An eBGP peer with iBGP peering with the reflector might prefer the iBGP route than the eBGP route.

## Route Reflector cluster

- On large designs there is need to use more than one Route Reflector
- RR clusters allow redundancy and hierarchy
- Cluster is defined by the clients as RR serves
- RRs in the same cluster use the same Cluster-ID
- Inter-Cluster peering can be between Clients or non-Client peerings and dependes on the topology and redundancy
- Cluster-ID is based on Router-ID by default
- Virtual Route Reflectors can be used as Reflector
    - A Virtual Route Relfector doesn't forward traffic
    - It is used only to reflect learned routes
    - Usually a VM with a lot of CPU and RAM

## BGP Confederations

- Disruptive change
- Reduce full mesh iBGP splitting AS into smaller sub-AS
- Inside Sub-AS full mesh or RR requirement remains
- Between Sub-AS act as eBGP
- Devices outside the confederation do not know about the internal structure. Sub-AS numbers are stripped from advertisements to true eBGP peers.
- Usually use ASNs in private range (64512 - 65535)
- Can use different IGPs in each Sub-AS
- Next-hop, Local-Pref, and MED are kept Sub-AS eBGP peerings.
- AS_CONFED_SEQUENCE and AS_CONFED_SET, never leave the confederation

## Design considerations  

### Route Reflectors vs Confederations

- Migrate to confederation is hard
- Greenfield confederation is easier
- Migrate to RR is easy, just add peers then remove old ones.

### Partial Mesh designs

- Full mesh, Route Reflectors and Confederations can interoperate
- Some portions of the network needs path diversity
- Inter-cluster Route Reflectos for scaling larger

## Cisco Configuration

(on RR)
router bgp [ASN]
    neighbor [peer's IP] route-reflector-client
    bgp cluster-id [cluster-id]

(confederations)
router bgp [sub-ASN]
    bgp confederation-id [main-ASN]
    bgp confederation-peers [sub-as1 sub-asn] (just those directly connected)

# TODO

eBGP peering points are used for traffic policy on behalf of the internal network
Selective RIB Download Feature prevents BGP paths from being installed in RIB/FIB for escalability.
address-family [ipv6] [unicast]


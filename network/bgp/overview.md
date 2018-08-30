# BGP Overview

- Based in open standar
- Path-vector exterior gateway protocol
- Use multiple "attributes" for routing decision
- The attributes are for a prefix, not for a link, so it is easier to make traffic engineering.
    - IGP has a visibility of the topology and decide on link attributes (cost belongs to the link) and traffic Engineering is hard with IGP for this reason
- The most scalable and extensible protocol 
- Extensible through Address Family Identifier and Sub-Address Family Identifier
    - Custom extension
    - IPv6 Unicast, IPv4 & IPv6 Multicast, IPv4 & IPv6 MPLS, VPLS.
- BGP is not a routing protocol per-se
    - BGP is an application
    - BGP is a reachability protocol
    - BGP (generally) cannot route the network alone
    - BGP (generally) relies on IGP for transport and recursion
    - BGP advertise the route details but not the path details
- BGP timers hold time 180 keep alive 60 by default
    - Keepalives are sent once the peers have converged and not before
    - Keepalive interval will be negotiated during the session establishment. Will use the shorter value.

## BGP use cases

- Provider Independent address
    - You own your addresses and BGP ASN
    - You dictate the policy
- Egress and Ingress policies
    - It is easy to influence egress traffic
        - I always choose how I route out
    - It is easy to influence ingress traffic
        - I (generally) choose how traffic returns
        - Sometimes difficutl on Internet scale
    - Do I really need the full view?
        - 500,000+ IPv4 prefixes
        - Does RIB have enough memory?
        - Can FIB actually install it at the linecard?
        - Is default OK?
- Scaling the Enterprise
    - Islands of IGP, Core of BGP
- Scaling DMVPN
    - Hubs as BGP Route Reflectors
    - BGP hierarchy is arbitrary
- Scaling Data Center Fabric
    - Use BGP for routing in large-scale data centers

## Autonomous System

A set of routers under a single technical administration, using an interior gateway protocol (IGP) and common metrics to determine how to route packests within the AS, and using inter-AS routing protocol to determine how to route packets to other ASes.

- Autonomous System Numbers (ASN) are asigned by the IANA
    - Public ASNs 1 - 64511
    - Private ASNs 64512 - 65535
    - 4 Byte ASN support is negotiated during capability exchange
        - 0.X denote original 2 byte ASN
        - Old BGP speakers are sent ASdot numbers encoded as ASN 23456
        - Real AS-Path encoded with optional transitive attributes AS4_AGGREGATOR and AS4_PATH

## BGP peering

Establishing BGP peering
    - BGP needes to find neighbors to exchange information with
    - BGP does not have its own transport
    - BGP has different types of neighbors
    - BGP neighbors are not discovered by default
    - BGP neighbors do not have to be directly connected
    - BGP uses TCP port 179 for transport (it implies that BGP needs IGP first)

Adding  neighbor tells process to
    - Listen for remote address via TCP 179
    - Initiate a session to remote address via TCP 179 (if configured as active)
    - BGP peer must agree on where the peering session is coming from (otherwise it will refuse the connection)
        - Can be modified with update-source per neighbor
    - There are two types of peering types:
        - External BGP (Between peers from different AS)
        - Internal BGP (Between peers from same AS)
    - Capabilities are exchanged when the peering goes up in the BGP Open message.
        - Some capabilities must be met in order to establish a BGP session

### eBGP peering

- TTL set to 1 by default. 
- TTL security is used to set the TTL to 255 and the receiving peer can check the receiving TTL has been decremented in the range expected.
- It is possible to use disable-connected-check in Cisco routers to don't decrement the TTL value when routing between router interfaces.
    - This is useful if the peers are directly connected but using the loopbacks for peering.
- eBGP uses the AS Path to avoid loops. It discards inbound updates containing local ASN in the AS Path.

### iBGP peering

- As good practie use loopback as the source interface to maintain the connection if an interface goes down
    - Some MPLS L3VPN require loopback as the source interface
- TTL set to 255 by default (use IGP to reach the peer)
- iBGP learned routes cannot be advertised to another iBGP peer
    - Fully mesh iBGP peering
    - Route reflectors
    - Confederations

## Next-Hop

- BGP next-hop controls IGP route recursion
- BGP knows the next-hop but not the outgoing interface
- IGP must be used to perform recursion otherwise the route can't be used
- If next-hop recursion fails, the route is not installed in the RIB
- Different peerings have different next-hop processing rules
- It is recommended to advertise the ISPs directed connected links in IGP
- iBGP updates won't modify the next-hop value regardless of the peering type
    - iBGP is not in charge of routing the traffic, the IGP is
    - Useful for route reflectors
    - It can be modified if required
        - next-hop-self
        - route-map
        - It is possible to lose fast convergence when the next-hop is modified
- eBGP outbound updates have local update-source for the neighbor set as next-hop (i.e lo0)
    - It assumes the peer doesn't know about our internal topology
    - It can be modified if required
        - route-map
        - neighbor next-hop-unchanged
        - Limited corner case applications

## Capabilities

Some important BGP-4 capabilities

- Route refresh
    - Re-advertisement of the Adj-RIB-Out from a BGP peer
- Enhaced Refresh Capability
    - Provide for the demarcation of the beginning and the ending of a route refresh
- Four octect ASN capability
    - Use 4 bytes for the ASN instead of 2 bytes.
    - AS4_PATH and AS4_AGGREGATOR used to propagate four-octet AS numbers when a speaker doesn't support the capability
- Outbound Route Filtering
    - Allows a BGP speaker to send to its BGP peer a set of Outbound Route Filters (ORFs)
    - The peer would then apply these filters, in addition to its locally configured outbound filters
- Graceful restart
    - Ability to preserve forwarding state during BGP restart
    - Temporarily retaining routing information across a TCP session termination/re-establishment

## Cisco Commands

router bgp [local ASN]
  neighbor [peer's IP] remote-as [peer's ASN]
  neighbor [peer's IP> update-source [source interface]
  neighbor [peer's IP] transport connection-mode [active | passive]
  neighbor ebgp-multihop [ttl]
  neighbor ttl-security hops [ttl]
  disable-connected-check (to don't decrement the TTL in the same's router interfaces)
  timers bgp [keep-alive] [hold time]
  network [prefix] mask [mask]


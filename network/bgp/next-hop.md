# BGP Next-Hop

BGP is not a routing protocol per se
    - It is an application used to exchange NLRI
    - Includes the route detail, but not the path details
- IPv4 NLRI contains
    - Prefix, prefix length
    - Attributes
        - Local preference
        - AS Path
        - Med
  - Next-Hop

- BGP Next-Hop needs IGP route recursion
    - BGP knows the next-hop but not the outgoing interface
    - IGP must be able to perform recursion otherwise the route can't be used
- If next-hop recursion fails the route won't be installed in the RIB or advertised
- Different peerings have different nex-hop processing rules

Usually there are three solutions for the next-hop from eBGP to an internal AS
- Advertise the link where the next-hop is into the IGP
- In the router with eBGP peering (also with iBGP peering):
    - Change the next-hop (next-hop-self to update next-hop with the source peering interface address) 
    - Change the next-hop manually (set next address with a route-map)

## Third Party Nex Hop

- Used when multiple router are connected on a segment, same IP same subnet
- Not all routers form eBGP peering.
    - Hub and spoke design
        - Hub router doesn't change next hop IP address when sending eBGP updates on same segment
    - Internet exchange

## Set Next-Hop to self

- Pros:
    - Peer can use the same next-hop on outbound updates to iBGP peers
    - Result is same dynamic update-group
    - Don't need to include external link in IGP
- Cons:
    - Hinders fast convergence of external uplink failure
    - It is possible to link the convergence of BGP to the IGP using a next-hop tracking, usually the prefix length of the recursive route.
    - If external links are unstable, can cause churn in IGP
    - Type-5 LSAs are flooded to all non-stub areas


## Cisco Commands

neighbor [peer address] next-hop-self


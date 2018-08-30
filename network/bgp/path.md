# BGP Best Path Selection

Chooses a single BGP best-path which can be
- Installed in the RIB/FIB
- Advertised to other BGP peers
- Standard selection of path among vendors (mostly)

## Prerequisites

- Next-hop value must be in the routing table
    - Prevents route recursion failure
- Synchronization rule must be met or disabled
    - Legacy black-hole prevention technique
    - Disabled by default
- AS-Path must not contain local-AS
    - Normal eBGP loop prevention
    - Can be disabled wit _allow-as in_
- First ASN in the path must be neihbor's ASN
    - _bgp enforce-fir st-as_ command

## Best Path Selection Order

- Pre-Bestpath community
- Weight
    - Cisco propietary
    - Locally significant
    - Higher value is preferred
- Local preference
    - Well-known discretionary
    - Higher value is preferred
    - Not advertised to eBGP peers
    - Carried through confederation eBGP
- Locally Originated
    - Locally originated gets Weight of 32768
- AS-Path
    - Well-known mandatory
    - Smaller length is preferred
    - Disable with _bgp best as-path ignore_
- Origin
    - Well-known mandatory
    - IGP over EGP over Incomplete
- Multi-Exit Discriminator
    - Optional non-transitive
    - Smaller value is preferred
    - Only compared between for peerings to same provider by default
    - Not standarized among vendors 
- eBGP over iBGP
    - If you learned if via eBGP it's not your prefix
- IGP Metric to Next-Hop
    - Can use multi-path if all equal after this step
    - _bgp bestpath igp-metric ignore_
- Tie breakers
    - Oldest
    - Lowest RID
    - Shorter cluster list
    - Lowest Neighbor Address

Multipath load balancing for external links with enequal bandwidth capacity
- Enable under IPv4, IPv6, VPNv4, VRF AF
- For iBGP, eBGP, eiBGP
- Still only one best path advertised to peers

## Manipulating Best Path Selection

- Longest Match Routing is above all
- Affects both directions

### Influence inbound traffic

- Outbound routing policy affects inbound traffic
- Set outbound
- Affects inbound traffic
- Attributes to influence Inbound Path Selection
    - AS-Path
        - Pre-pend own ASN
        - It is possible to block an AS prepending its ASN in local advertisments
        - Only possible on eBGP peering
    - MED
        - Because is non-transitive it has to be configured in eBGP peers to have effect on remote AS
        - Not a good method to influence traffic
        - Only used when peering to the same AS

### Influence outbound traffic

- Inbound routing policy affects outbound traffic
- Set inbound
- Affects outbound traffic
- Attributes to Influence Outbound Path Selection
    - Weight
        - First check if the next-hop can route the prefix
        - Even it has locally significance, it can influence other peers (e.g. setting the weight in a RR)
        - Bad practice
    - Local Preference
        - Affects all routers in an AS
        - Non-transitive attribute

### Communities

- Transitive
- Not a negotiated capability
- Used for extensibilty
- Outbound community signaling affects outbound policy on the other side
- Can be used to set attributes on the remote ASN
    - Usually agreed on in a legal document (pre-defined communities)
    - Set local preference
        - [ASN]:[Local Preference]
    - Set other attributes

- BGP Cost-community
    - BGP Custom Decision process
    - Only advertised within the AS or confederation peers
    - Influence BGP path selection at the Point of Insertion
        - "pre-bestpath" point of insertion can be used
        - Compare cost-community value before weight

## Aggregation

- Summarization can apply anywhere
    - BGP hierarchy is arbitrary
    - Path vector is like distance vector
    - You only learn what your neighbor tells you
        - Unlike OSPF and IS-IS

### Options to aggregate prefixes

- Summary route
    - Only Requirement is that a subnet must exist in BGP table
    - Advertise the aggregate and the individual prefixes by default
    - Set atomic-aggregate attribute 
    - By default it will delete the AS-Path
    - _aggregate-address [network] [mask] [args]_
        - summary-only
            - suppress more specific routes
    - suppres-map
        - route-map to tell what specific routes to suppress
    - as-set
        - Un-order list of ASN and communities are associated with the aggretate
        - Inherit from more specific routes
    - advertise-map
        - attributes of prefix or prefixes matched with advertised-map are inherited in aggregate address
        - Used in conjunction with as-set
            - Prevents a specific AS from being included in the AS_SET of the aggregate
            - Circunvetns loop prevention
            - _ip as-path access-list [number] permit [as regex]_
    - attribute-map | route-map
        - used to modify attributes associated with aggregate address
        - used in conjunction with as-set
    - It is possible to leak a more specific prefix for traffic engineering
        - Has to be done in the router aggregating the prefixes
        - It is done per neighbor basis
        - The outbound is processed first and then the unsuppred routes
            - Use the same unsuppresed route-map to set attributes
        - neighbor [ip peer] unsuppress-map [route-map]

- auto-summary
    - Not recommended
    - Disabled by default
    - Classful boundaries
    - No mask required on network statement
    - Requires a subnet of network to be present in IGP

## Cisco Commands

(communities)
    - neighbor [IP address] send-community
    - network [prefix] mask [mask] route-map [route-map]
    - ip bgp-community new-format
    - set community [community] additive
    - ip community-list standard [community] permit [community]
    - match community [community-list]


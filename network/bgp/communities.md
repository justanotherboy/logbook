# BGP Communities

It is a optional transitive attribute
    - Defined in RFC 1997
    - A community is like a route-tag
    - Not exchanged between peers by default
    - neighbor [IP address] send-community
    - Standard community is 4-byte value
        - Decimal (0 - 2^32)
        - 2 byte format: AA:NN

It can be used to:
    - Apply a policy
    - Filter traffic
    - Traffic engineering

## Well-Known Communities

- NO_EXPORT (0xFFFFFF01)
    - Not advertise outside the AS
- NO_ADVERTISE (0xFFFFFF02)
    - Not advertise to other BGP peers
- NO_EXPORT_SUBCONFED (0xFFFFFF03)
    - Not advertise to eBGP (including sub-AS in a confederation)

## Use of Communities

- Set occurs directly in route-map
    - set community {community-number [additive] [well-known-community] | none }
    - Can apply the route-map to the NLRI   
    - Not "additive" by default
        - Will overwrite the communities
    - Can delete a community

- Match occurs via community-list
    - Define list
        - Standard list matches community name or number
            - _ip community-list 1 standard permit no-export_
        - Expanded matches regular expression
            - _ip community-list expanded AS100 permit 100:[0-9]+_
    - Reference from route-map
        - _match community AS100_
    
## Extended Communities

- Defined RFC 4360
- Used for extended applications such as
    - MPLS L3VPN Route Target
    - MPLS L2VPN
    - EIGRP cost community

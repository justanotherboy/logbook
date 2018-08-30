# BGP Network Layer Reachability Information

Network Layer Reachability Information
    - Route with its prefix
    - BGP uses UPDATE and WITHDRAW messages to exchange NLRI

## NLRI Origination

### Network statement

- Requires exact match in the routing table first
- Originates prefixes with ORIGIN of IGP (i)
- Without _mask_ keyword, assume classful mask
- Sets weight to 32768 (to prefer own origination)

### Redistribute statement

- Won't include OSPF External by default
    - redistribute ospf [pid] match internal external
- Originates prefixes with ORIGIN INCOMPLETE
- Originates classful summary if auto-summary is enable
- Automatically copies IGP metric to BGP MED
- Sets weight to 32768 (to prefer own origination)
    - Aggregate-address statement
        - Require one subnet in BGP table first
    - BGP inject-map statement
        - Opposite of aggretation
        - Originates subnets from aggregate for purpose of longest match (traffic engineering)
        - inject-map inject-map exist-map exist-map [copy-attributes]

## BGP conditional Advertisement

- neighbor advertis-map map1 [non-exist-map map2 | exist-map map2 ]
- Advertise prefix matched in advertise-map:
    - if prefix matched in non-exist-map does not exist
    - if prefix matched in exist-map does exist
- Typically used to track failure of a transit link
    - E.g. Advertise to backup provider, only if primary provider is down
- Sometimes is slow to converge

## Redistribution to IGP

- By default only external BGP routes are redistribute into IGP
    - bgp redistribute internal
    - can cause a loop


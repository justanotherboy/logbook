# OSPF Path Selection

Every router LSAs include cost attribute for each link. Cost is an arbitrary number but most implementations use a relation of bandwidth. The SPF algorithm calculates the shortest path to other routers (and their prefixes) in the same area. To reach inter-area routes an ABR is used, the ABR will hide the topology details and there is no need to calculate SPF for inter-area routes.

## Path Selection

Prefered order to install a route in to the RIB:
1. Intra-area route
2. Inter-area route
3. Intra-area external route
    - E1, N1, E2, N2
4. Inter-area external route
    - E1, N1, E2, N2

It is possible to select a Nx over Ex if the forwarding metric is less for the Nx. This is the case for routers which have the LSA 5 and 7 to the same prefix.

### Path Cost for external routes

- Intra-area external routes:
    - ASBR can reach prefix A in cost X
    - Local router can reach ASBR via SPT in cost Y
    - Local router can reach prefix A via SPT in cost X + Y

- Inter-area external routes:
    - ASBR can reach link A in cost X
    - ABR can reach ASBR via SPT in cost Y
    - Local router can reach ABR via SPT in cost Z
    - Local router can reach link A via SPT in cost X + Y + Z

Traffic Engineering in stub area:
- Set Area Default Cost in the ABR
- Longest Match Routing

### LSA recursion

For LSA type 5:

(ASBR is in the same area) LSA 5 -> LSA 1
(ASBR is not in the same area) LSA 5 -> LSA 4 -> LSA 3 -> LSA 1
(ASBR is not in the same area and ASBR is in a NSSA) LSA 5 -> LSA 3 -> LSA 1

When recursing from 5 to 4 the router-id is used
When recursing from 5 to 3 the IP address (Forward Address) is used, the IP address should be in the routing table.

For LSA type 7:

(ASBR is in the same area) LSA 7 -> LSA 1

## Summarization

OSPF can do summarization in two ways:
- Per-prefix summarization
    - Replace multiple longer matches with a shorter match
    - Prefix summarization will create a discard route by default. The goal is to drop traffic if longest match is a summary.
        - A summary route cannot fallback to default.
    - Stub areas

- Per-LSA summarization:
    - Remove all Inter-Area routes and replace them with a the shortes match possible
    - Stub areas

All devices in same area must have the same LSDB
- Implies summarization can't be performed at arbitrary points
- Summarization can happen between internal areas or between external domains

Internal summarization
- Summarizes Type-1 into Type-3
- Only at the ABR who knows the LSA-1. Summarize LSA-1 to LSA-3 but not LSA-3 to LSA-3

External summarization
- Summarize Type-5 into Type-5 LSAs
- Summarize Type-7 into Type-7 LSAs
- Only at the ASBR who is the originator

Other summary applications
- Traffic Engineering
    - Prefer longer match
- Filter routes
- Enforce area-local scope for NSSA routes


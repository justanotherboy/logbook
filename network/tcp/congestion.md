# Congestion Control

Congestion control is a set of behaviors determined by algorithms that each TCP implements in an attempt to prevent the network from being overwhelmed by aggregated traffic load. The basic approach is to have TCP slow down when it has reason to believe the network is about to be (or is already) congested.

When a router is forced to discard data because it cannot handle the arriving traffic rate, is called congestion. Even a single connection can drive one or more routers into congestion. Congestion can cause the performance of a network to be reduced so badly that it becomes unusable.

## Detection of Congestion in TCP

There is no explicit signaling about congestion. Instead, TCP needs to conclude that congestion is occurring to be able to react. In TCP, an assumption is made that a lost packet is an indicator of congestion, and that some response (i.e., slowing down) is required. Other methods for detecting congestion, including measuring delay and network-supported Explicit Congestion Notification (ECN) allow TCP to learn about congestion before packet loss.

The window size field in the TCP header is used to signal a sender to adjust its window based on the buffer at the receiver. The sender should slow down if the receiver is too slow or if the network is too slow. This is accomplished by introducing a window control variable at the sender that is based on an estimate of the network's capacity. A sending TCP then sends at a rate equal to what the receiver or the network can handle, whichever is less.

```
W = min(cwnd, awnd)

W = Sender's actual window
cwnd = Congestion window
awnd = Receiver's advertised window
```

The TCP sender is not permitted to have more than W unacknowledged bytes outstanding in the network. This is sometimes called the flight size. The "correct" value for cwnd is not directly available to the sending TCP. Thus, all values must be empirically determined and dynamically updated. W should be set about the bandwidth-delay product (BDP) of the network path, also called the optimal window size. This is the amount of data that can be stored in the network in transit to the receiver. It is equal to the product of the RTT and the capacity of the lowest capacity link on the path.



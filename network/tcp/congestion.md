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

## Classic Algorithms

When a new TCP connection starts out, it usually has no idea what the initial value for cwnd should be. The way to learn a good value for cwnd is to try sending data at faster and faster rates until it experiences a congestion indicator (usually a packet drop). To avoid negative effects on the performance of other TCP connections, TCP uses one algorithm to avoid starting so fast when it starts up to get to steady state. It uses a different one once it is in steady state.

The operation of TCP congestion control at a sender is driven by the receipt of ACKs. If a TCP is operating at steady state, receipt of an ACK indicates that one or more packets have been removed from the network, and consequently that an opportunity to send more has arisen. The TCP congestion behavior in steady state attempts to achieve conservation of packets in the network.

TCP uses slow start and congestion avoidance algorithms for congestion control. TCP executes only one at any given time, but it may switch back and forth between the two.

### Slow Start

The slow start algorithm is executed when a new TCP connection is created, when a loss has been detected due to a retransmission timeout, or when TCP has gone idle for some time. The purpose of slow start is to help TCP find a value for cwnd before probing for more available bandwidth using congestion avoidance and to establish the ACK clock. Typically, a TCP begins a new connection in slow start, eventually drops a packet, and then settles into steady-state operation using congestion avoidance.

A TCP begins in slow start by sending a certain number of segments, called the initial window (IW):
* IW = 2(SMSS) and not more than 2 segments (if SMSS > 2190 bytes)
* IW = 3(SMSS) and not more than 3 segments (if 2190 > SMSS > 1095 bytes)
* IW = 4(SMSS) and not more than 4 segments (otherwise)

Slow start operates by incrementing cwnd by min(N, SMSS) for each good ACK received, where N is the number of previously unacknowledged bytes ACKed by the received "good ACK". Eventually, cwnd (and thus W) could become so large that the corresponding window of packets sent overwhelms the network. When this happens, cwnd is reduced substantially (to half of its former value). In addition, this is the point at which TCP switches from operating in slow start to operating in congestion avoidance. The switch point is determined by the relationship between cwnd and a value called slow start threshold (ssthresh).

### Congestion Avoidance

Slow start increases cwnd fairly rapidly and helps to establish a value for ssthresh. Once this is achieved, there is always the possibility that more network capacity may become available. To find additional capacity that may become available, TCP implements the congestion avoidance algorithm, which increases cwnd by approximately one segment for each window's worth of data that is moved from the sender to receiver successfully. cwnd is usually updated as follows for each received nonduplicate ACK:

```
cwndₜ₊₁ = cwndₜ + SMSS * SMSS/cwndₜ 
```

Congestion avoidance grows the window linearly (also called additive increase), whereas slow start grows it exponentially.

### Selecting between Slow Start and Congestion Avoidance

A TCP connection is always running either the slow start or the congestion avoidance procedure, but never the two simultaneously. The ssthresh is a limit on the value of cwnd that determines which algorithm is in operation. When cwnd < ssthresh, slow start is used, and when cwnd > ssthresh, congestion avoidance is used. The value of ssthresh is not fixed but instead varies over time. It remembers the last "best" estimate of the operating window when no loss was present. The initial value of ssthresh may be set arbitrarily high. When a retransmission occurs, caused by a retransmission timeout or fast retransmit, ssthresh is updated as follows:

```
ssthresh = max(flight size/2, 2(SMS))
```

TCP assumes that the operating window must have been too large for the network to handle. Reducing the estimate of the optimal window size is accompanied by altering ssthresh to be about half of what the current window size is. This usually results in lowering ssthresh, but it can also result in increasing ssthresh.

## Tahoe

The slow start and congestion avoidance, constitute the first congestion control algorithms applied to TCP. They were introduced in the 4.2 release of BSD (called Tahoe). The release included a version of TCP that started connections in slow start, and if a packet was lost, detected by either a timeout or the fast retransmit, the slow start algorithm was reinitiated. Tahoe reduced the cwnd to its starting value (1 SMSS) upon any loss, forcing the connection to slow start until cwnd grew to the ssthresh value.

## Reno

One problem setting the cwnd value to 1 is that for large BDP paths, the connection significantly underutilizes the available bandwidth while the sending TCP goes through slow start. To address this problem, the reinitiation of slow start on any packet loss was reconsidered. If a packet loss is detected by duplicate ACKs, cwnd is instead reset to the last value of ssthresh instead of 1 SMSS and enters in the fast recovery phase. This approach allows the TCP to slow down to half its previous rate without reverting to slow start. TCP Reno became very popular and ultimately the basis for what might be called "standard TCP"

### Fast recovery

Fast recovery allows cwnd to temporarily grow by 1 SMSS for each ACK received while recovering. Any nonduplicate ACK causes TCP to exit recovery and reduces the congestion window back to its pre-inflated value.

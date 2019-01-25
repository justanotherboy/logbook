# TCP Retransmissions

TCP has two different mechanisms to initiate retransmissions, one based on timeouts and other based on inferring losses by the acknowledgment structure. TCP sets a timer when it sends data if the timer expires and the data is not acknowledged a timer-based retransmission occurs. A fast retransmission happens without a delay when the cumulative acknowledgment doesn't advance or when receiving a segment with a SACK block.

## Basic concepts

### Retransmission Timeout (RTO)

TCP keeps track of the Round-Trip Time (RTT) using the sequence numbers and acknowledgment numbers. The challenge is to set a good  RTO estimate based on the RTT measurements. Setting the RTO too low would generate spurious retransmissions, setting it too high would underutilize the network.

### Clock granularity

The TCP clock doesn't have infinite precision and doesn't reflect the real system clock. Usually, the TCP clock is a variable that is updated as the system clock advances not necessarily one-to-one. The time it takes to the TCP clock to advance is called its granularity. Modern implementations use a clock granularity in the order of milliseconds or tens of milliseconds.

### Karn's Algorithm

The retransmission ambiguity problem is as follow:
The sender retransmits a segment and receives the acknowledgment. Is the acknowledging the original packet? Is for the retransmitted segment? The Karn's algorithm solves the problem for the RTO calculation and involves two steps:
1. Ignore the acknowledgment of a retransmitted packet for RTO calculations.
2. Apply a backoff factor for each subsequent retransmission. Double the RTO every time a segment is retransmitted. Return the RTO to normal when the sender receives an acknowledgment for a non-retransmitted packet.

### Round-Trip Time Calculation Using Timestamps TCP Option (TSOPT)

The TSOPT allows measuring the RTT continuously. The sender sets its TCP clock in the TSV field the receiver echoes this value in the TSER field for packets which contain data (segments with the SYN and FIN flags count as data). The sender can measure the RTT using its current TCP clock and the TSER in the received packet. The process seems simple enough but is not as straightforward. There might be packet loss, reordering, and retransmissions, also TCP usually acknowledge every other segment. Most implementations use the following algorithm to calculate the RTT:
1. The sender sets its TCP clock in the TSV field of the TSOPT in every TCP segment.
2. The receiver keeps track of the most recent TSV value to send in the next acknowledgment.
3. Whenever a new in-order segment arrives, the TSV value to send is updated.
4. When the receiver generates an acknowledgment with TSOPT the receiver set the TSER value to the last valid TSV from the sender.
5. The sender receiving the acknowledgment substracts the TSER from its TCP clock to calculate the RTT.

The TSOPT algorithm works even in cases of reordering and retransmissions:
- **Out-of-order segments:** When a receiver receives an out-of-order segment it will generate an acknowledgment immediately to trigger the fast retransmit at the sender. This ACK includes as its TSER value the TSV of the most recent in-order segment. This usually causes the sender to increase the RTT measure, therefore increase its RTO.
- **Retransmission:** When a receiver receives a retransmitted segment the acknowledge generated includes as its TSER value the TSV of the retransmitted segment. This causes the RTT value on the sender to go down, therefore decrease its RTO.

When using the TSOPT to calculate the RTT there is no need to implement the first step of the Karn's algorithm.

## Methods to calculate the RTO

### RFC 793 method to set the RTO

The RFC 798 uses a smoothed round-trip time estimator (SRTT) to calculates the retransmission timeout:

```
SRTT = α(SRTT) + (1-α)RTTs
```

The recommended value for α is a value between 0.8 and 0.9. The newly estimated consists of 80% to 90% from the previous estimate and 10% to 20% of the new sample. This average is also called Exponentially Weighted Moving Average (EWMA). Given the SRTT the recommended RTO is set to the following:

```
RTO = min(ubound, max(lbound, β(SRTT))
```

β is a delayed variance factor recommended between 1.3 to 2.0. ubond is the upper bound (suggested to 60 seconds), and lbound is the lower bound (recommended to 1 second). For a stable RTT this method works fine, but not for networks with highly variable RTT such as wireless networks.

### RFC 6298 method to calculate the RTO

For networks with high variability on the Round-Trip Time, it is possible to track an estimate of the RTT variability (RTTVAR) for a better calculation of the RTO. The mean deviation is a good enough standard deviation approximation and it is faster to calculate (because mean deviation doesn't require square root). The calculations are as follows:

```
SRTT = (1-α) SRTT + α (RTTs)
RTTVAR = (1 - β) RTTVAR + β * | SRTT - RTTs |
RTO = max(SRTT + max(G, 4 * RTTVAR), 1000)

α = 1/4
β = 1/8
G = Clock granularity
```

This method uses a smoothed SRTT and RTTVAR which varies over time. For TCP timer based retransmissions this algorithm or another less aggressive must be implemented as stated by the RFC 6298. Please note most TCP implementations doesn't set the RTO floor to one second.

#### Initial Values

There is no information to set the RTO before initiating the connection (unless there is cached information in the system). Per the RFC 6298, the initial value for the RTO should be 1 second, 3 seconds when the first SYN is retransmitted. The endpoint set the following values once it receives the first data:

```
SRTT = RTTs
RTTVAR = RTTs/2
```

### Linux method to calculate the RTO

Linux uses a clock granularity of 1 millisecond and uses TSOPT for RTO calculation. Such small granularity and constant RTT measures, make the RTO calculations more accurate; however, the RTTVAR tends to be minimized over the time. Changes in the last RTT sample would increase the RTTVAR even when the RTT decreases. Linux keeps four variables to deal with cases of high variability between RTT samples: SRTT, RTTVAR, MDEV, MDEV_MAX.  MDEV_MAX is the maximum MDEV seen in the last measured RTT  and can't be less than 50 ms. In case the new RTT sample is less than the current smoothed RTT Linux give less weight to the last estimation.

```
MDEV = (1 - β) RTTVAR + β * | SRTT - RTTs |
MDEV_MAX = max(50ms, max(MDEV_MAX, MDEV))
RTTVAR = MDEV_MAX
SRTT = (1-α) SRTT + α (RTTs)
RTO = SRTT + 4*RTTVAR

α = 1/4 if (RTTs < (SRTT - MDEV))
α = 1/8 if (RTTs > (SRTT - MDEV))
β = 1/8
```

## Timer-based retransmission

Once TCP has established the RTO based upon measurements of RTT, whenever TCP sends a segment it also set the retransmission timer. When the endpoint receives the acknowledgment on-time for a packet, TCP cancels the retransmission timer. When TCP fails to receive an acknowledgment within the RTO, TCP performs timer-based retransmission. When there is a retransmission the sender will reduce the sending window accordingly to its congestion control algorithm and will start with the 2nd step of the Karn's algorithm; the RTO is temporarily multiplied by a backoff value when multiple retransmissions occur for the same segment.

## Fast retransmit

Fast retransmit can trigger packet retransmission without waiting for the retransmission timer to expire. Usually, this method is more efficient and faster than timer-based retransmissions. When an out-of-order segment is received, the endpoint will generate an immediate acknowledgment (a duplicate ACK) and the sender's needs to fill the hole at the receiver as quickly and efficiently as possible.

A duplicated acknowledgment at the sender is an indicator packet was lost or delayed. Because it is not possible to know which one, the sender waits for a small number of duplicate ACKs before initiating a fast retransmit. Some implementations wait for a small number of duplicated acknowledgment (usually 3) while others base the number on the current level of reordering. Packet loss is inferred because of duplicate ACKs and congestion control measures are invoked too.

## Retransmission with Selective Acknowledgments

With the SACK option, the receiver is able to signal data that has been received beyond the cumulative ACK. Not contiguous data are called out-of-sequence data and there are holes between contiguous and non-contiguous data.

The sender needs to fill the holes by retransmitting any data the receiver is missing without sending duplicate data. A SACK capable receiver can include up to four holes of data (three if using TSOPT). The sender is able to fill these holes more quickly than a non-SACK sender.

### SACK Receiver Behavior

The receiver places in the first SACK block the sequence number range contained in the segment it has most recently received to provide the most recent information to the sender. Other SACK blocks are listed in the order in which they appeared as first blocks in the previous SACK options to provide some redundancy in the case where SACKs are lost.

### SACK Sender Behavior

The sender must perform selective retransmission by only sending those segments missing at the receiver. When a SACK-capable sender performs a retransmission, it has the choice of whether it sends new data or retransmits old data. The simplest approach is to have the sender first fill the hole and then move on to send more new data if the congestion control procedures allow. This is the most common approach.

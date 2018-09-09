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

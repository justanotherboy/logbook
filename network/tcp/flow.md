# Data Flow

Studies of TCP traffic usually find that 90% or more of all TCP segments contain bulk data (e.g., web, file sharing) and the remaining portion contains interactive data (e.g., remote login). Bulk data segments tend to be relatively large, while interactive data segments tend to be much smaller. TCP handles both types of data using the same protocol and packet format, but different algorithms come into play for each.

## Delayed Acknowledgments

Using a cumulative ACK allows TCP to intentionally delay sending an ACK for some amount of time, in hope that it can combine the ACK it needs to send with some data the local application wishes to send in the other direction. This form of piggybacking that is used most often in conjunction with bulk data transfers.

The Host Requirements RFC (RFC 1122) states that TCP should implement a delayed ACK but the delay must be less than 500ms. Many implementations use a maximum of 200ms. Delaying ACKs causes less traffic to be carried over the newtork because fewer ACKs are used. A ratio of 2 to 1 is fairly common for bulk transfers. TCP is generally set up to delay ACKs under certain circumstances, but not to delay them too long.

## Nagle Algorithm

A small packet (called tinygram) have a relatively high overhead for the network. They contain relatively little useful application data compared to the rest of the packet contents.

The Nagle algorithm says that when a TCP connection has outstanding data that has not yet been acknowledged, small segments (smaller than the SMSS) cannot be sent until all outstanding data is acknowledged. Instead, small amounts of data are collected by TCP and sent in a single segment when an acknowledgment arrives. The faster the ACKs come back, the faster the data is sent, the RTT controls the packet sending rate.

### Delayed ACK and Nagle Algorithm Interaction

Generally, TCP is required to provide an ACK for two received packets only if they are full-size. The combination of delayed ACKs and the Nagle algorithm leads to a form of deadlock. Fortunately, this deadlock is not permanent and is broken when the delayed ACK timer fires. However, the entire data transfer becomes idle during the deadlock period. The Nagle algorithm can be disabled in such circumstances.

## Flow Control

Every TCP segment (except SYN segments) includes a valid Sequence Number, ACK Number, and a Window Size field. The window size field represents the amount of space the sender of the segment has reserved for storing incoming data. TCP-based applications are typically able to consume all data TCP has received and queued for them, leading to no change of the Window Size field. On slow systems, or when the application has other things to accomplish, data may have arrived for the application, been acknowledged by TCP, and be sitting in a queue waiting for the application to consume it. When TCP starts to queue data in this way, the amount of space available to hold new incoming data decreases, and this change is reflected by a decreasing value of the Window Size field. If the application does not consume the data at all, the Window Size will set to zero (no space).

### Sliding Windows

For each TCP connection, each TCP endpoint maintains a send window structure and a receive window structure. The window advertised by the receiver is called the offered window. The Window Size field contains a byte offset relative to the ACK number. The sender computes its usable window, which is how much data it can send immediately. The usable window is the offered window minus the amount of data already sent but not yet acknowledged.

        ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
        │      │      │      │      │      │      │      │      │      │      │      │      │
        │ ...  │  2   │  3   │  4   │  5   │  6   │  7   │  8   │  9   │  10  │  11  │  ... │
        │      │      │      │      │      │      │      │      │      │      │      │      │
        └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

        │<-  Sent and ACK  ->│<-Sent and not ACK->│<-    Begin Sent  ->│<-Can't send until->│
                                                      (Usable Window)       window moves

                             │                                         │
                             └── Closes ─>                <─ Shrinks ──┴── Opens ─>

                         Left Edge                                Right Edge

Over time this sliding window moves to the right, as the receiver acknowledges data. The relative motion of the two ends of the window increases or decreases the size of the window. Three terms are used to describe the movement of the right and left edges of the window:

* The window closes as the left edge advances to the right. This happens when data that has been sent is acknowledged and the window size gets smaller.
* The window opens when the right edge moves to the right, allowing more data to be sent. This happens when the receiving process consumes data, freeing up space in its TCP receive buffer.
* The window shrinks when the right edge moves to the left. The Host Requirements RFC strongly discourages this, but TCP must be able to cope with it.

The left edge of the window cannot move to the left, because the cumulative acknowledge never goes backward. When an ACK advances the window but the window size does not change, the window is said to advance or slide forward. If the ACK number advances but the window advertisement grows smaller with other arriving ACKs, the left edge moves closer to the right edge. If the left edge reaches the right edge, it is called a zero window. This stops the sender from transmitting any data.

With selective ACKs, other in-window segments can be acknowledged, but ultimately the ACK number itself is advanced only when data contiguous to the left window edge is received.

### Zero Windows

When the receiver's advertised window goes to zero, the sender is effectively stopped from transmitting data until the window becomes nonzero. When the receiver once again has space available, it provides a window update to the sender to indicate that data is permitted to flow once again. Because such updates do not generally contain data, they are not reliably delivered by TCP. If an acknowledgment (containing a window update) is lost, we could end up with both sides waiting for the other. To prevent this form of deadlock from occurring, the sender uses a persist timer to query the receiver periodically, to find out if the window size has increased. The persist timer triggers the transmission of window probes. Window probes contain a single byte of data and are therefore reliably delivered by TCP, thereby eliminating the potential deadlock condition. As with the TCP retransmission timer the normal exponential backoff can be used when calculating the timeout for the persist timer.

### Silly Window Syndrome (SWS)

The silly window syndrome is when small data segments are exchanged across the connection instead of full-size segments, therefore each segment has high overhead. SWS can be caused by either end of a TCP connection: the receiver can advertise small windows (instead of waiting until a larger window can be advertised), and the sender can transmit small data segments (instead of waiting for additional data to send a larger segment). The following rules are applied to avoid the silly window syndrome:

* When operating as a receiver, small windows are not advertised. The algorithm specified by RFC1122 is to not send a segment advertising a larger window than is currently being advertised (which can be 0) until the window can be increased by either one full-size segment or by one-half of the receiver's buffer space, whichever is smaller.
* When sending, small segments are not sent and the Nagle algorithm governs when to send. Senders transmit if at least one of the following condition is true:
    - A full-size segment can be sent.
    - TCP can send at least one-half of the maximum-size window that the other end has ever advertised on this connection.
    - TCP can send everything it has to send when:
        * An ACK is not currently expected
        * The Nagle algorithm is disabled for this connection.

Multiplexing   → Many application communications share transport/network services.

Demultiplexing → Transport layer delivers received data to the correct application/socket using port information.


Layer 4 — Transport Layer

Main question:

Once the packet reaches the correct computer, which application/process should receive the data?

That is Layer 4’s job.

Core idea

Suppose your laptop is running:

Chrome
WhatsApp
SSH client
Spotify

All are using the same machine IP.

So IP address alone tells us:

“Send this to this computer.”

But not:

“Send this specifically to Chrome.”

For that, we use:

Port Numbers

Example:

HTTPS → 443
HTTP  → 80
SSH   → 22
DNS   → 53

So Layer 4 provides:

Process-to-process delivery

PDU

For TCP:

Segment

For UDP:

Datagram

Exam shortcut:

TCP → Segment
UDP → Datagram
Main protocols
Transport Layer
│
├── TCP
└── UDP
1. TCP

TCP =

Transmission Control Protocol

TCP is:

Connection-oriented
Reliable
Ordered
Error-controlled
Flow-controlled

Think:

“I want the data to arrive correctly and in order.”

Examples where TCP is commonly used:

HTTP/HTTPS
FTP
SSH
Email protocols
TCP 3-Way Handshake

Before data transfer:

Client                 Server

SYN
---------------------->

        SYN + ACK
<----------------------

ACK
---------------------->

Then connection is established.

Exam sequence:

SYN
SYN-ACK
ACK

Very important.

Why TCP is reliable

Suppose sender sends:

Segment 1
Segment 2
Segment 3

TCP uses:

Sequence Numbers
Acknowledgements
Retransmission

If Segment 2 is lost:

1 ✅
2 ❌
3 ✅

TCP can retransmit Segment 2.

So TCP gives:

Reliable delivery.

Ordered delivery

Suppose segments arrive:

3
1
2

TCP can reorder them:

1
2
3

before passing them to the application.

ACK

ACK =

Acknowledgement

Receiver tells sender:

“I received the data.”

If ACK does not arrive within expected time, sender may retransmit.

Flow Control

Flow control means:

Prevent a fast sender from overwhelming a slow receiver.

Example:

Sender capacity   = very fast
Receiver capacity = slow

TCP manages this using a:

Window mechanism

For now just remember:

TCP Flow Control → Sliding Window

We don’t need deep window calculations yet.

Error Control

TCP detects problems and can recover through:

Checksum
ACK
Retransmission
Sequence numbers

Again, first-pass level only.

2. UDP

UDP =

User Datagram Protocol

UDP is:

Connectionless
Faster
Lower overhead
No delivery guarantee
No ordering guarantee
No retransmission mechanism like TCP

It just sends.

Think:

“Speed is more important than perfect delivery.”

Examples:

DNS
VoIP
Live streaming
Online gaming

Though modern applications can use different protocols internally, these are good exam associations.

TCP vs UDP
TCP	UDP
Connection-oriented	Connectionless
Reliable	Unreliable/best effort
Ordered delivery	No ordering guarantee
More overhead	Less overhead
Slower comparatively	Faster comparatively
Uses ACKs	No TCP-style ACKs
Retransmission	No built-in retransmission
Segment	Datagram

Very important for IBPS.

Port Numbers

Port number size:

16 bits

Therefore range:

0 to 65535

Important ranges:

0–1023
→ Well-known ports

1024–49151
→ Registered ports

49152–65535
→ Dynamic / Private / Ephemeral ports

For exam purposes, definitely remember:

Well-known ports → 0–1023
Source Port vs Destination Port

Suppose:

Your browser → Google HTTPS

Your system may choose:

Source Port = 53124
Destination Port = 443

So:

Your Laptop
53124
   ↓
Google
443

When Google responds:

Source Port = 443
Destination Port = 53124

So port numbers reverse in the reply.

Multiplexing / Demultiplexing

One useful Layer-4 concept:

Your computer can have many applications using the network simultaneously.

Example:

Chrome    → Port connection A
Spotify   → Port connection B
SSH       → Port connection C

Transport layer uses port numbers to separate them.

This is called:

Multiplexing
Demultiplexing

Simple meaning:

Many applications
      ↓
same network stack
      ↓
port numbers keep them separate
Segmentation

Suppose application gives:

Large data

Transport layer may divide it into smaller pieces:

Segment 1
Segment 2
Segment 3

Receiver reassembles them.

So:

Segmentation
Reassembly

are important Transport-layer functions.

Transport Layer Functions
Transport Layer
│
├── Process-to-process delivery
├── Port addressing
├── Segmentation
├── Reassembly
├── Flow control
├── Error control
├── Reliability
└── Multiplexing

Not every function is provided equally by both TCP and UDP.

One exam trap

Layer 3:

Host-to-host delivery
→ IP

Layer 4:

Process-to-process delivery
→ Port

So:

IP tells:
Which computer?

Port tells:
Which application?
Full connection so far
Application Data
      ↓
Transport Layer
Src Port + Dst Port
      ↓
Network Layer
Src IP + Dst IP
      ↓
Data Link
Src MAC + Dst MAC
      ↓
Physical
Bits

This is now the complete lower-stack picture.
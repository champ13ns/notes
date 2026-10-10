A router has to receive a Layer-2 frame first, remove that Layer-2 encapsulation, inspect the Layer-3 packet, make a routing decision, and then create a new Layer-2 frame for the next link.


Layer 3 — Network Layer
Main job

Layer 3 mainly handles:

Logical Addressing
Routing
Path Selection
Packet Forwarding

PDU -> Packet.

Address used -> IP Address.

Main Device -> Router.

A Layer-3 switch can also perform routing, but for exam purposes:
Router = Layer 3 device

Why do we need Layer 3?
Suppose:
Laptop
IP = 192.168.1.10

wants to reach:

Google
IP = 142.x.x.x

They are not on the same local network.

A switch cannot solve this problem by itself because a normal switch works mainly using MAC addresses inside its local Layer-2 network.

We need a router.

Laptop
   ↓
Switch
   ↓
Router
   ↓
Internet
   ↓
Google

┌─────────────────────────────┐
│ IP Header                   │
│                             │
│ Source IP      = Your IP    │
│ Destination IP = Google IP  │
├─────────────────────────────┤
│ TCP Header                  │
│ Src Port = 53124            │
│ Dst Port = 443              │
├─────────────────────────────┤
│ Data                        │
└─────────────────────────────┘

Now it is called a packet.


MAC addrss is generally called as physical address.
IP Address is called as logical address.

Destination IP
      ↓
Routing Table
      ↓
Choose next hop/interface
      ↓
Forward packet


Default Gateway -> used when the desitination is outside the local network.

Same Network vs Different Network

Suppose:

PC-A = 192.168.1.10
PC-B = 192.168.1.20

Assume they are in the same subnet.

Then:

A → Switch → B

No router is required for communication between them.

But suppose:

PC-A = 192.168.1.10
PC-B = 192.168.2.10

and these are different networks.

Now:

A → Router → B

A router is needed to move packets between networks.

Exam line:

Router connects different networks.


Routing -> Selecting a path to reach another network
Types -> Static Routing, Dynamic Routing.

Static Routing -> Administrator manually configure routes. To reach network X, send via Router B.
Dynamic Routing -> Router learn/updates using routing protocols -> Important ones are
RIP -> Distance Vector
OSPF -> Link State
BGP -> Path Vector.

ICMP

ICMP belongs to the Network layer / Internet layer.

Full form:

Internet Control Message Protocol

Used for:

Error reporting
Diagnostics
Network control messages

Famous example:

ping

Ping commonly uses:

ICMP Echo Request
ICMP Echo Reply

So exam trap:

Does ping use TCP or UDP?

Neither.

It uses ICMP.


NAT

Because your laptop may have:

192.168.1.10

Google cannot directly route to that private address.

Your router commonly performs:

NAT — Network Address Translation

Conceptually:

192.168.1.10
      ↓ NAT
Public IP
      ↓
Internet

Where does ARP fit?

You already know this.

Suppose Layer 3 decides:

Next hop IP = 192.168.1.1

But Layer 2 needs:

Destination MAC = ?

ARP helps find:

IP → MAC

Example:

192.168.1.1
      ↓ ARP
AA:BB:CC:DD:EE:FF

So mentally:

Routing tells us:
"WHO is the next hop IP?"

ARP tells us:
"What is that next hop's MAC?"


ARP -> Job is to convert MAC Address from IP Address
10.10.10.2
      ↓ ARP
AA:BB:CC:DD:EE:22


Switch Table vs ARP Table vs Routing Table

Swithc Table -> Mac Addresss -> Port
ARP Table -> IP Address -> MAC Address.
Routing Table -> IP Adrress -> next hop ip.



NAT — Network Address Translation

So unlike ordinary routing, NAT actually modifies the IP header.

Why is NAT needed?

Mainly because IPv4 public addresses are limited.

Suppose you have:

Laptop
Phone
TV
Tablet
Desktop

All can have private IPs:

192.168.1.10
192.168.1.11
192.168.1.12
192.168.1.13
192.168.1.14

but all may share one public IP:

49.36.20.50
But then how does the router know who gets Google's response?

This brings us to:

PAT

PAT =

Port Address Translation

Also called:

NAT Overload

Suppose:

Laptop:
192.168.1.10:50001

Phone:
192.168.1.11:50002

Router can translate:

192.168.1.10:50001
      ↓

49.36.20.50:61001

and:

192.168.1.11:50002
      ↓

49.36.20.50:61002

Router keeps a NAT table:

Public                 Private
-----------------------------------------
49.36.20.50:61001  → 192.168.1.10:50001

49.36.20.50:61002  → 192.168.1.11:50002

When Google responds:

49.36.20.50:61001

router knows:

Send to 192.168.1.10:50001

So one public IP can serve many internal devices.

NAT vs PAT

For IBPS:

NAT
→ Translates IP addresses
PAT
→ Translates IP + Port
→ Allows many private hosts to share one public IP
NAT -> Network address translation -> Private IP -> Public IP

PAT -> Port Address translation -> Public IP -> Private IP



So:

SWITCH
→ Reads L2 frame
→ Forwards FRAME
→ Doesn't normally decapsulate to Layer 3

Whereas:

ROUTER
→ Receives frame
→ Removes L2 encapsulation
→ Processes IP PACKET
→ Creates new L2 frame

This switch-vs-router difference is the exact point to lock in.


Your flow should be:

Router receives packet at Layer 3:

[Source IP = Original Sender]
[Destination IP = Final Receiver]
[Payload]

Router sees:

Destination IP != my own IP

So it checks the routing table:

Destination IP
     ↓
Best route
     ↓
Next-hop IP / Outgoing Interface

Suppose:

Next-hop IP = 10.0.0.2

Now router needs the next hop's MAC.

10.0.0.2
   ↓ ARP
MAC-R2

At this point the router knows:

Next-hop IP  = 10.0.0.2
Next-hop MAC = MAC-R2
Outgoing interface = Interface X

Then the IP packet is handed down to Layer 2.

Layer 2 creates:

Source MAC
= MAC address of THIS router's outgoing interface

Destination MAC
= MAC address of next-hop router

So the new frame becomes:

┌───────────────────────────────┐
│ L2 Header                     │
│                               │
│ Src MAC = R1 outgoing MAC     │
│ Dst MAC = R2 MAC              │
├───────────────────────────────┤
│ IP Packet                     │
│                               │
│ Src IP = Original sender      │
│ Dst IP = Final destination    │
├───────────────────────────────┤
│ L2 Trailer / FCS              │
└───────────────────────────────┘

So if:

PC-A → R1 → R2 → Google

then between R1 and R2:

Layer 3:

Src IP = PC-A
Dst IP = Google

Layer 2:

Src MAC = R1's outgoing interface MAC
Dst MAC = R2's interface MAC

Then at R2, this Layer-2 header is removed again.

R2 gets the same IP packet:

Src IP = PC-A
Dst IP = Google

checks routing table, finds R3, gets R3's MAC using ARP if needed, and creates another new frame:

Src MAC = R2
Dst MAC = R3

So the clean rule is:

Layer 3 decides:
"Where should this packet go next?"

Layer 2 decides/builds:
"How do I frame it for that next-hop link?"

And yes, the source MAC also changes, because the new frame is now being transmitted by the router's outgoing network interface.

One tiny correction too: destination IP does not have to be different from the router's IP for routing logic in every case; if it is one of the router's own IPs, the packet is for the router itself. If it isn't, the router may forward it according to the routing table.
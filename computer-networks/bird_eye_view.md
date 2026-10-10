COMPUTER NETWORKS
│
├── 1. NETWORKING BASICS 
│   │
│   ├── What is a Computer Network
│   │   ├── Interconnection of devices
│   │   ├── Communication between systems
│   │   └── Resource sharing / data sharing
│   │
│   ├── Need / Advantages of Networking
│   │   ├── File sharing
│   │   ├── Printer / hardware sharing
│   │   ├── Communication
│   │   └── Centralized management
│   │
│   ├── Types of Networks
│   │   ├── PAN → Personal Area Network
│   │   ├── LAN → Local Area Network
│   │   ├── MAN → Metropolitan Area Network
│   │   └── WAN → Wide Area Network
│   │
│   ├── Network Topologies
│   │   ├── Bus
│   │   ├── Star
│   │   ├── Ring
│   │   ├── Mesh
│   │   ├── Tree
│   │   └── Hybrid
│   │
│   └── Transmission Modes
│       ├── Simplex
│       ├── Half Duplex
│       └── Full Duplex
│
├── 2. OSI MODEL & TCP/IP MODEL 
│   │
│   ├── OSI Model (Layer) (PDU -> Protocol data unit)
│   │   ├── Physical (Layer 1)  (Bits) -> Common devices -> Hub, Repeater, Modem, Cables
│   │   ├── Data Link (Layer 2) (Frames) -> It uses mac addressing to identify hosts , Common devices -> Bridge, Switches
│   │   ├── Network  (Layer 3) (Packets) -> Assigns unique IP addresses (sender and receiver) to the packet header to identify devices globally. Common devices -> Routers, Switches.
│   │   ├── Transport (Layer 4)  (Segments/datagram) -> Uses Port Numbers (e.g., Port 80 for web traffic) to ensure data is delivered to the specific process or application intended, not just the device. Protocols : (TCP, UDP)
│   │   ├── Session (data)
│   │   ├── Presentation (data)
│   │   └── Application (data)
│   │
│   ├── TCP/IP Model
│   │   ├── Network Access / Link
│   │   ├── Internet
│   │   ├── Transport
│   │   └── Application
│   │
│   ├── OSI vs TCP/IP
│   │   ├── 7 layers vs 4 layers
│   │   ├── Reference model vs protocol suite
│   │   └── Mapping of layers
│   │
│   ├── Encapsulation / Decapsulation
│   │   ├── Data
│   │   ├── Segment
│   │   ├── Packet
│   │   ├── Frame
│   │   └── Bits
│   │
│   └── Addressing across layers
│       ├── Port Number → Transport
│       ├── IP Address → Network
│       └── MAC Address → Data Link
│
├── 3. PHYSICAL LAYER BASICS 
│   │
│   ├── Signals & Transmission Concepts
│   │   ├── Analog vs Digital
│   │   ├── Bandwidth
│   │   ├── Throughput
│   │   ├── Latency
│   │   └── Noise
│   │
│   ├── Transmission Media
│   │   ├── Guided Media
│   │   │   ├── Twisted Pair
│   │   │   ├── Coaxial Cable
│   │   │   └── Optical Fiber
│   │   │
│   │   └── Unguided Media
│   │       ├── Radio Waves
│   │       ├── Microwaves
│   │       └── Infrared
│   │
│   └── Important points
│       ├── Fiber → highest speed / long distance
│       ├── Twisted pair → cheap / common LAN
│       └── Coaxial → better shielding than twisted pair
│
├── 4. DATA LINK LAYER & MAC 
│   │
│   ├── Data Link Layer Functions
│   │   ├── Framing
│   │   ├── Physical addressing
│   │   ├── Error detection
│   │   ├── Flow control
│   │   └── Access control
│   │
│   ├── Framing
│   │   ├── Fixed size
│   │   ├── Variable size
│   │   └── Frame boundaries
│   │
│   ├── Error Detection
│   │   ├── Parity
│   │   ├── Checksum
│   │   └── CRC
│   │
│   ├── Flow Control
│   │   ├── Stop-and-Wait idea
│   │   └── Prevent sender from overwhelming receiver
│   │
│   ├── MAC Addressing
│   │   ├── 48-bit physical address
│   │   ├── Hexadecimal form
│   │   └── Used in local network delivery
│   │
│   ├── ARP 
│   │   ├── IP → MAC resolution
│   │   ├── Broadcast ARP Request
│   │   └── Unicast ARP Reply
│   │
│   └── Exam traps
│       ├── MAC changes hop-to-hop? → next-hop delivery uses MAC
│       ├── IP remains end-to-end
│       └── ARP works in local network
│
├── 5. CHANNEL ACCESS / MULTIPLE ACCESS 
│   │
│   ├── ALOHA
│   │   ├── Pure ALOHA
│   │   └── Slotted ALOHA
│   │
│   ├── CSMA
│   │   ├── Sense before transmit
│   │   ├── 1-persistent
│   │   ├── Non-persistent
│   │   └── p-persistent
│   │
│   ├── CSMA/CD
│   │   ├── Collision Detection
│   │   ├── Used in traditional Ethernet
│   │   └── Wired networks
│   │
│   ├── CSMA/CA
│   │   ├── Collision Avoidance
│   │   ├── Used in Wi-Fi
│   │   └── Wireless networks
│   │
│   └── Key distinction
│       ├── CD → detect after collision
│       └── CA → avoid before collision
│
├── 6. ETHERNET, SWITCHING, VLAN, STP 
│   │
│   ├── Ethernet
│   │   ├── IEEE 802.3
│   │   ├── Ethernet Frame
│   │   ├── Source MAC
│   │   ├── Destination MAC
│   │   ├── Type/Length
│   │   ├── Data
│   │   └── FCS
│   │
│   ├── Switching Basics
│   │   ├── Switch works at Data Link layer
│   │   ├── Uses MAC table
│   │   ├── Learns source MACs
│   │   └── Forwards intelligently
│   │
│   ├── Hub vs Switch
│   │   ├── Hub → broadcast to all
│   │   ├── Switch → sends to correct port
│   │   ├── Hub → one collision domain
│   │   └── Switch → separate collision domain per port
│   │
│   ├── VLAN
│   │   ├── Logical segmentation
│   │   ├── One physical switch, multiple logical LANs
│   │   ├── Better security
│   │   └── Better broadcast control
│   │
│   ├── Trunking
│   │   ├── Carry multiple VLANs
│   │   └── VLAN tagging idea
│   │
│   ├── STP
│   │   ├── Spanning Tree Protocol
│   │   ├── Prevents switching loops
│   │   ├── Root Bridge
│   │   ├── Root Port
│   │   ├── Designated Port
│   │   └── Blocking port
│   │
│   └── Exam traps
│       ├── VLAN reduces broadcast domain size
│       ├── Switch breaks collision domains
│       └── STP prevents broadcast storm / loops
│
├── 7. WIRELESS NETWORKS & SHORT-RANGE TECH 
│   │
│   ├── Wi-Fi
│   │   ├── IEEE 802.11
│   │   ├── WLAN technology
│   │   └── Uses CSMA/CA
│   │
│   ├── Bluetooth
│   │   ├── IEEE 802.15.1
│   │   ├── Short-range communication
│   │   └── PAN use case
│   │
│   ├── Zigbee
│   │   ├── IEEE 802.15.4
│   │   ├── Low power
│   │   └── IoT / sensor networks
│   │
│   └── Key comparison
│       ├── Wi-Fi → more speed
│       ├── Bluetooth → short range
│       └── Zigbee → low power / low data rate
│
├── 8. NETWORK LAYER & IP ADDRESSING 
│   │
│   ├── Functions of Network Layer
│   │   ├── Logical addressing
│   │   ├── Routing
│   │   ├── Path determination
│   │   └── Packet forwarding
│   │
│   ├── IPv4 Basics
│   │   ├── 32-bit address
│   │   ├── Dotted decimal notation
│   │   └── Network ID + Host ID
│   │
│   ├── IP Address Classes
│   │   ├── Class A
│   │   ├── Class B
│   │   ├── Class C
│   │   ├── Class D
│   │   └── Class E
│   │
│   ├── Public vs Private IP
│   │   ├── 10.0.0.0/8
│   │   ├── 172.16.0.0 – 172.31.255.255
│   │   └── 192.168.0.0/16
│   │
│   ├── Subnetting 
│   │   ├── Borrow host bits
│   │   ├── Network portion increases
│   │   ├── Host portion decreases
│   │   └── Used to create smaller networks
│   │
│   ├── CIDR 
│   │   ├── Slash notation
│   │   ├── /24, /20, /29 etc.
│   │   └── Classless addressing
│   │
│   ├── VLSM 
│   │   ├── Variable Length Subnet Mask
│   │   ├── Different subnet sizes
│   │   └── Efficient IP allocation
│   │
│   ├── Supernetting 
│   │   ├── Route aggregation
│   │   └── Combine smaller blocks
│   │
│   └── Key skills
│       ├── Find subnet mask
│       ├── Find network ID
│       ├── Find host range
│       └── Find broadcast address
│
├── 9. ICMP & DIAGNOSTIC TOOLS 
│   │
│   ├── ICMP
│   │   ├── Error reporting
│   │   ├── Diagnostic messages
│   │   └── Works with IP
│   │
│   ├── Ping
│   │   ├── Uses ICMP
│   │   ├── Tests reachability
│   │   └── Measures RTT roughly
│   │
│   ├── Traceroute / Tracert
│   │   ├── Finds path to destination
│   │   ├── Uses TTL concept
│   │   └── Shows intermediate hops
│   │
│   └── TTL
│       ├── Prevents infinite looping
│       ├── Decrements at each router
│       └── 0 → packet discarded
│
├── 10. TRANSPORT LAYER 
│   │
│   ├── Functions of Transport Layer
│   │   ├── End-to-end delivery
│   │   ├── Segmentation / reassembly
│   │   ├── Flow control
│   │   ├── Error control
│   │   └── Process-to-process delivery
│   │
│   ├── Port Numbers
│   │   ├── Identify application process
│   │   ├── Well-known ports
│   │   └── Multiplexing / demultiplexing
│   │
│   ├── TCP
│   │   ├── Connection-oriented
│   │   ├── Reliable
│   │   ├── Sequencing
│   │   ├── Acknowledgement
│   │   ├── Retransmission
│   │   └── Flow control
│   │
│   ├── UDP
│   │   ├── Connectionless
│   │   ├── Unreliable
│   │   ├── Faster / lightweight
│   │   └── No handshake
│   │
│   ├── TCP 3-Way Handshake
│   │   ├── SYN
│   │   ├── SYN-ACK
│   │   └── ACK
│   │
│   ├── Reliability Concepts
│   │   ├── Sequence numbers
│   │   ├── ACK
│   │   ├── Timeout
│   │   └── Retransmission
│   │
│   └── TCP vs UDP
│       ├── TCP → reliable but heavier
│       └── UDP → fast but unreliable
│
├── 11. APPLICATION LAYER PROTOCOLS 
│   │
│   ├── DNS 
│   │   ├── Domain name to IP resolution
│   │   ├── Recursive query
│   │   ├── Iterative / referral idea
│   │   ├── Root server
│   │   ├── TLD server
│   │   └── Authoritative server
│   │
│   ├── DHCP 
│   │   ├── Dynamic IP assignment
│   │   ├── DORA Process
│   │   │   ├── Discover
│   │   │   ├── Offer
│   │   │   ├── Request
│   │   │   └── Acknowledge
│   │   └── APIPA idea
│   │
│   ├── HTTP / HTTPS 
│   │   ├── HTTP → application protocol
│   │   ├── HTTPS → HTTP over TLS/SSL
│   │   ├── Secure communication
│   │   └── Web browsing
│   │
│   ├── FTP
│   │   ├── File transfer
│   │   └── Client-server model
│   │
│   ├── Email Protocols
│   │   ├── SMTP → sending mail
│   │   ├── POP3 → download mail
│   │   └── IMAP → sync/manage mail
│   │
│   └── Common Ports 
│       ├── HTTP → 80
│       ├── HTTPS → 443
│       ├── FTP → 21
│       ├── SSH → 22
│       ├── Telnet → 23
│       ├── SMTP → 25
│       ├── DNS → 53
│       └── DHCP → 67/68
│
├── 12. NETWORK DEVICES 
│   │
│   ├── Repeater
│   │   ├── Regenerates signal
│   │   └── Physical layer
│   │
│   ├── Hub
│   │   ├── Multiport repeater
│   │   └── Broadcasts everywhere
│   │
│   ├── Bridge
│   │   ├── Connects LAN segments
│   │   └── Filters by MAC
│   │
│   ├── Switch
│   │   ├── Multiport bridge
│   │   └── Data Link layer
│   │
│   ├── Router
│   │   ├── Connects different networks
│   │   └── Uses IP routing
│   │
│   ├── Gateway
│   │   ├── Protocol conversion
│   │   └── Higher-layer functionality
│   │
│   ├── Modem
│   │   ├── Modulator / Demodulator
│   │   └── Digital ↔ analog conversion
│   │
│   └── Access Point
│       ├── Wireless connectivity
│       └── Connects wireless clients to LAN
│
├── 13. ROUTING & ROUTING PROTOCOLS 🟡
│   │
│   ├── Routing Basics
│   │   ├── Forwarding packets
│   │   ├── Path selection
│   │   └── Routing table
│   │
│   ├── Static Routing
│   │   ├── Manually configured
│   │   └── Good for small/stable networks
│   │
│   ├── Dynamic Routing
│   │   ├── Automatically learns routes
│   │   └── Good for larger/changing networks
│   │
│   ├── Routing Protocol Categories
│   │   ├── Distance Vector
│   │   ├── Link State
│   │   └── Path Vector
│   │
│   ├── RIP
│   │   ├── Distance vector
│   │   ├── Hop count metric
│   │   └── Max hop count = 15
│   │
│   ├── OSPF
│   │   ├── Link state
│   │   ├── SPF / Dijkstra idea
│   │   └── Faster convergence than RIP
│   │
│   ├── BGP
│   │   ├── Path vector
│   │   ├── Inter-AS routing
│   │   └── Internet backbone routing
│   │
│   └── Key comparison
│       ├── RIP → simple but slower / limited
│       ├── OSPF → scalable within AS
│       └── BGP → between autonomous systems
│
└── 14. REMAINING EXTENSION TOPICS ⬜
    │
    ├── NAT
    │   ├── Private to public conversion
    │   └── IPv4 conservation
    │
    ├── IPv6
    │   ├── 128-bit address
    │   ├── Huge address space
    │   └── Colon-hex notation
    │
    ├── Network Security Basics
    │   ├── Firewall
    │   ├── VPN
    │   ├── IDS / IPS
    │   └── Proxy
    │
    └── QoS / Congestion basics
        ├── Traffic handling
        └── Performance optimization
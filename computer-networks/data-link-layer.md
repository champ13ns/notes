Layer 2 - Data Link Layer., PDU - Frame

If we already have IP addresses at Layer 3, why do we also need MAC addresses at Layer 2?

Because IP tells us the final destination, while MAC tells us where to send the frame on the current local hop.

Laptop → Router 1 → Router 2 → Router 3 → Google

MAC addressing is hop-to-hop. IP addressing is end-to-end.

What does Data Link Layer do?

Layer 2 mainly provides:

Data Link Layer
│
├── Framing
├── MAC addressing
├── Error detection
├── Media access control
└── Node-to-node delivery

Addres : MAC Address.


Main devices:

Switch
Bridge
NIC 

The Layer 2 header contains things like:

Source MAC
Destination MAC

and the trailer commonly contains error-detection information such as:

FCS

FCS = Frame Check Sequence.


2. MAC Address

A MAC address is generally:

48 bits

Example:

A4:5E:60:12:34:56

48 bits =

6 bytes

and because every hex digit represents 4 bits:

48 bits ÷ 4 = 12 hexadecimal digits

So an exam question may ask:

Length of a MAC address?

Answer:

48 bits or 6 bytes.






DATA LINK LAYER
Layer        → 2
PDU          → Frame
Address      → MAC Address
Delivery     → Node-to-node

Devices:
→ Switch
→ Bridge
→ NIC

Functions:
→ Framing
→ MAC addressing
→ Error detection 
→ Media access control

MAC Address:
→ 48 bits
→ 6 bytes

Switch:
→ Maintains MAC/CAM table
→ Learns from SOURCE MAC
→ Forwards using DESTINATION MAC
→ Unknown destination → Flooding

Hub:
→ Multiport repeater

Switch:
→ Multiport bridge

Collision Domain:
Hub    → One
Switch → One per port

Broadcast Domain:
Normal switch → One
Router → Separates broadcast domains

Ethernet:
→ IEEE 802.3
→ CSMA/CD

Wi-Fi:
→ IEEE 802.11
→ CSMA/CA




When we say:

MAC AA:AA:AA → Port 1
MAC BB:BB:BB → Port 2

here Port 1, Port 2 mean the physical switch interfaces:

Switch
│
├── Ethernet Port 1 → PC-A
├── Ethernet Port 2 → PC-B
├── Ethernet Port 3 → Router
└── Ethernet Port 4 → Printer

So remember:

Transport Port
→ Logical number
→ Identifies application/process
→ 80, 443, 22, 53, etc.

Switch Port
→ Physical/logical switch interface
→ Where a cable/device is connected
→ Port 1, Port 2, Gi0/1, Gi0/2, etc.

Same word port, totally different meaning.
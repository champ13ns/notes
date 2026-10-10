p d n t s p a



| Protocol    | Purpose               | Port |
| ----------- | --------------------- | ---: |
| FTP Data    | File transfer data    |   20 |
| FTP Control | File transfer control |   21 |
| SSH         | Secure remote login   |   22 |
| Telnet      | Remote login          |   23 |
| SMTP        | Send email            |   25 |
| DNS         | Name resolution       |   53 |
| DHCP Server | IP configuration      |   67 |
| DHCP Client | IP configuration      |   68 |
| TFTP        | Simple file transfer  |   69 |
| HTTP        | Web                   |   80 |
| POP3        | Receive mail          |  110 |
| NTP         | Time synchronization  |  123 |
| IMAP        | Mail access           |  143 |
| SNMP        | Network management    |  161 |
| SNMP Trap   | Notifications         |  162 |
| HTTPS       | Secure web            |  443 |
| LDAP        | Directory service     |  389 |
| RDP         | Remote desktop        | 3389 |


20/21 → FTP
22    → SSH
23    → Telnet
25    → SMTP

| Protocol | Main job                        | Important mapping/port |
| -------- | ------------------------------- | ---------------------- |
| DNS      | Domain name resolution          | Name → IP, Port 53     |
| DHCP     | Automatic network configuration | UDP 67/68              |
| ARP      | Resolve local/next-hop MAC      | IPv4 → MAC             |
| ICMP     | Diagnostics/error reporting     | Ping, no TCP/UDP port  |

If DHCP fails, common APIPA range?
169.254.0.0/16.


Layer 1, -> Hub, Repeater, Modem
Layer 2, -> Switch, Bridge, NIC
Layer 3 -> Router



| Standard      | Technology             |
| ------------- | ---------------------- |
| IEEE 802.3    | Ethernet               |
| IEEE 802.11   | Wi-Fi / WLAN           |
| IEEE 802.15   | WPAN                   |
| IEEE 802.15.1 | Bluetooth              |
| IEEE 802.15.4 | Zigbee / low-rate WPAN |

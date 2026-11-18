# Réseau - Notes

## TryHackMe - What is Networking? (Pre Security > Network Fundamentals)

### Task 1 - What is a network?
- A network is simply things (devices) connected together.
- Networks are everywhere in daily life (metro, electricity grid, mail...).
- A network can be made of 2 devices or billions of them.

### Task 2 - What is the Internet?
- The Internet is one giant network made of many smaller networks interconnected.
- ARPANET (late 1960s) was the first documented network in action.
- The World Wide Web was invented in 1989 by Tim Berners-Lee.
- Private networks are connected together through the Internet (a public network).

### Task 3 - Identifying devices on a network
Two identifiers for a **device** (not the network itself):

**IP address (Internet Protocol)**
- Can change (assigned by the network, e.g. via DHCP).
- IPv4: 4 sections called octets, e.g. `192.168.1.10` -> 32 bits, 2^32 ≈ 4.29 billion addresses.
- IPv6: 8 hexadecimal groups, e.g. `2001:db8::1` -> 128 bits, 2^128 ≈ 3.4 × 10^38 addresses. Created because IPv4 addresses are running out.
- Private IP: used inside a local network. Public IP: given by the ISP, used on the Internet.
- **NAT**: all devices of a private network share the same public IP when talking to the Internet (done by the router / box).

**MAC address (Media Access Control)**
- Burned into every network card, globally unique in principle.
- 12 hexadecimal characters (48 bits), e.g. `a4:c3:f0:85:ac:2d`.
- First 6 characters = manufacturer (OUI), last 6 = unique ID of the network card.

**Practical exercise - hotel Wi-Fi**
Alice paid for Internet, Bob didn't. By changing Bob's MAC address to Alice's, Bob gets access too.
-> This is **MAC spoofing**.

### Task 4 - Ping
- Ping uses **ICMP** (Internet Control Message Protocol).
- Sends an *ICMP Echo Request*, the target answers with an *Echo Reply*.
- Used to check if a device is reachable and to measure round-trip time (latency).
- Syntax: `ping 10.10.10.10` or `ping example.com`
- On Linux, ping runs until stopped: `ping -c 4 10.10.10.10` sends only 4 packets.

### Pourquoi c'est important en sécu
- A MAC address can be changed easily -> never use it alone to authenticate someone (MAC filtering is weak).
- Ping (ICMP) is often the first step of reconnaissance to discover live hosts; many firewalls block it for that reason.
- Knowing private vs public IP and NAT is essential to understand how an attacker reaches (or can't reach) internal machines.

## TryHackMe - Intro to LAN (Pre Security > Network Fundamentals)

### Task 1 - LAN topologies
A topology = how the devices of a network are physically/logically connected.

| Topology | How it works | Pros | Cons |
| --- | --- | --- | --- |
| Star | Every device connected individually to a central device (switch or hub) | Easy to add devices, easy troubleshooting (isolate one device) | Expensive (cabling + equipment), central device = single point of failure |
| Bus | All devices on one backbone cable (like leaves on a branch) | Cheap, easy to set up | Slow when traffic grows (one shared cable = bottleneck), hard to troubleshoot, backbone cut = whole network down |
| Ring (token ring) | Devices connected in a loop, data travels around until it reaches its destination | Less prone to bottlenecks, a device sends its own data first | One cable/device failure breaks the whole loop |

### Network devices
- **Switch**: connects many devices (PCs, printers...) of the same local network, usually via Ethernet. Forwards frames to the right port using **MAC addresses** (layer 2).
- **Hub**: older device, sends every frame to all ports (no intelligence).
- **Router**: connects different networks and passes data between them = **routing**. Uses **IP addresses** (layer 3). Chooses the best path when several paths exist (redundancy).
- Switches and routers are combined: switches build the LAN, the router connects it to other networks (e.g. the Internet).

### Questions
- LAN = Local Area Network
- Job of a router = routing
- Device that centrally connects devices and sends data to the right place = switch
- Cost-efficient topology = bus
- Expensive topology = star

### Pourquoi c'est important en sécu
- Single point of failure (star central switch, bus backbone) = risk on **availability** (the A of the CIA triad: Confidentiality, Integrity, Availability).
- On a hub, every device receives all traffic -> anyone can sniff it. Switches reduce this, but attacks like MAC flooding or ARP spoofing can make a switch leak traffic.

### Task 2 - Subnetting
- Subnetting = dividing a network into smaller networks (subnets), like cutting a cake into slices.
- A **subnet mask** (32 bits, 4 octets from 0 to 255) splits an IP address into a **network part** and a **host part**.
- Example: `192.168.1.0/24` (mask `255.255.255.0`):
    - Network address (start of the network): `192.168.1.0`
    - Host addresses (devices): `192.168.1.1` to `192.168.1.254` -> 254 usable hosts
    - Broadcast address (send to everyone in the subnet): `192.168.1.255`
    - Default gateway: the router's address (often `.1` or `.254`), used to reach other networks
- Small networks (home) = usually one subnet. Companies use many subnets (per department, servers, guests...).

### Task 3 - ARP (Address Resolution Protocol)
- MAC address = physical identifier. IP address = logical identifier.
- ARP finds the **MAC address that matches an IP address** on the local network.
- **ARP Request** (broadcast to everyone): "Who has 192.168.1.5? Tell 192.168.1.10"
- **ARP Reply** (sent back to the asker): "192.168.1.5 is at aa:bb:cc:dd:ee:ff"
- The answer is stored for a while in the **ARP cache** (`arp -a` to view it).

### Task 4 - DHCP (Dynamic Host Configuration Protocol)
- DHCP automatically gives a device its network configuration: IP address, subnet mask, default gateway, DNS server, for a limited time (lease).
- 4 steps = **DORA**:
    1. **Discover** (device -> broadcast): "Is there a DHCP server?"
    2. **Offer** (server -> device): "You can use this IP"
    3. **Request** (device -> server): "I want this IP"
    4. **ACK** (server -> device): "Confirmed, it's yours"

### Questions (Tasks 2 to 4)
- Dividing a network = subnetting / bits in a subnet mask = 32 / range of an octet = 0-255
- Start of a network = network address / devices = host address / sends data to another network = default gateway
- ARP = Address Resolution Protocol / asks if a device has an IP = ARP Request
- Physical identifier = MAC / logical identifier = IP
- Packet to get an IP = DHCP Discover / sent after the offer = DHCP Request / last packet = DHCP ACK

### Pourquoi c'est important en sécu (Tasks 2 to 4)
- Subnetting enables **network segmentation**: if one subnet is compromised, the attacker can't easily reach the others (limits lateral movement).
- ARP has **no authentication**: an attacker can send fake ARP replies (**ARP spoofing**) to place himself between two machines = man-in-the-middle.
- DHCP can be abused too: a **rogue DHCP server** can give victims a fake gateway or DNS; **DHCP starvation** exhausts all available IPs.

## Professor Messer - Understanding the OSI Model (Network+ N10-009, 1.1)

### What is the OSI model?
- OSI = **Open Systems Interconnection** reference model.
- A framework that describes how data travels through a network, in **7 layers**. Each layer has its own role and protocols.
- Used every day to **troubleshoot** ("it's a layer 1 problem" = probably a cable) and to talk about equipment and attacks.
- Mnemonic from layer 7 to layer 1: **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

### The 7 layers

| # | Layer | Role | Data unit | Examples |
| --- | --- | --- | --- | --- |
| 7 | Application | Protocols used by applications to talk on the network (not the app itself) | Data | HTTP/HTTPS, DNS, FTP, SMTP, POP3 |
| 6 | Presentation | Formats the data: character encoding, compression, encryption/decryption | Data | TLS (SSL is the old, obsolete version), JPEG, ASCII/UTF-8 |
| 5 | Session | Opens, maintains and closes the communication (session) between two applications | Data | Control protocols, tunneling protocols (PPTP, L2TP) |
| 4 | Transport | Delivers data from one application to another (end to end), using **ports** | Segment (TCP) / Datagram (UDP) | **TCP** = reliable, connection (3-way handshake); **UDP** = fast, no delivery guarantee |
| 3 | Network | Routes traffic between different networks using **IP addresses** | Packet | IP, ICMP, routers |
| 2 | Data Link | Communication between devices of the **same local network** using **MAC addresses** ("switching layer") | Frame | Ethernet, MAC (EUI-48), switches, ARP |
| 1 | Physical | Transmission of the raw signal: electrical, light, radio | Bits | Cables, fiber, Wi-Fi signal, hubs |

- Problem with IP addresses or subnet masks -> layer 3. Problem with a cable or signal -> layer 1.
- In practice, layers 5, 6 and 7 are often grouped together (that's how the TCP/IP model sees them).

### Encapsulation
When sending, each layer adds its own header around the data from the layer above:
Data -> **Segment** (+ TCP/UDP header, ports) -> **Packet** (+ IP header, IP addresses) -> **Frame** (+ Ethernet header, MAC addresses) -> **Bits** on the wire.
The receiver does the opposite (decapsulation).

### Wireshark
- Wireshark captures network traffic and shows each packet **layer by layer** (Ethernet -> IP -> TCP/UDP -> application protocol): the OSI model in practice.

### Pourquoi c'est important en sécu
Attacks and defenses are described by layer:
- **L1**: physical access, cable tapping, Wi-Fi jamming.
- **L2**: ARP spoofing, MAC flooding (seen in Intro to LAN).
- **L3**: IP spoofing, ICMP abuse (ping sweeps for reconnaissance).
- **L4**: port scanning (Nmap), SYN flood (denial of service).
- **L7**: web attacks (SQL injection, XSS), phishing.
- A classic firewall filters at layers 3-4 (IP + port); a WAF (Web Application Firewall) filters at layer 7.

## Subnetting - Method and exercises

### Method (for /24 and above)
1. Host bits = 32 - prefix (e.g. /26 -> 6 host bits)
2. Block size = 2^(host bits) (e.g. 2^6 = 64)
3. Usable hosts = block size - 2 (network address + broadcast removed)
4. Network address = largest multiple of the block size <= last octet
5. Broadcast = network address + block size - 1
6. Mask last octet = 256 - block size (e.g. 256 - 64 = 192 -> 255.255.255.192)

### Exercises

| Address | Mask | Network | Broadcast | Usable hosts |
| --- | --- | --- | --- | --- |
| 192.168.1.77/24 | 255.255.255.0 | 192.168.1.0 | 192.168.1.255 | 254 |
| 192.168.1.77/26 | 255.255.255.192 | 192.168.1.64 | 192.168.1.127 | 62 |
| 172.16.5.130/25 | 255.255.255.128 | 172.16.5.128 | 172.16.5.255 | 126 |
| 192.168.10.45/27 | 255.255.255.224 | 192.168.10.32 | 192.168.10.63 | 30 |
| 10.0.0.200/28 | 255.255.255.240 | 10.0.0.192 | 10.0.0.207 | 14 |

### Why it matters
- Subnetting tells whether two machines are on the same network:
    - same subnet -> direct communication through the switch (ARP, layer 2)
    - different subnet -> traffic goes through the default gateway / router (layer 3)
- Pentest: after getting an IP, compute its subnet to know which range to scan (`nmap 192.168.1.64/26`).
- SOC / firewall rules and logs use CIDR notation (/24, /26...) everywhere.

## Lab - My first Wireshark capture (home Wi-Fi)

### My network
- `192.168.1.254` = Freebox = default gateway (Wireshark shows "FreeboxSas" thanks to the MAC OUI `38:07:16`).
- `192.168.1.8` = my Mac. MAC `f2:33:bd:66:d6:cf` = **randomized private MAC** (first octet `f2` -> "locally administered" bit set). macOS uses a different MAC per Wi-Fi network to prevent tracking.
- My Mac also has a **public IPv6** (`2a01:e0a:...`, a Free prefix): with IPv6 there is no NAT, every device gets its own global address.

### ARP (filter `arp`)
- Freebox: "Who has 192.168.1.8? Tell 192.168.1.254" -> my Mac: "192.168.1.8 is at f2:33:bd:66:d6:cf".
- My Mac: "Who has 192.168.1.29?" sent to **Broadcast** (MAC unknown), no reply -> device off or asleep.
- Ethernet type `0x0806` = ARP. In hex, `c0 a8 01 fe` = 192.168.1.254.

### DNS (filter `dns`)
- My Mac asks the Freebox (`192.168.1.254`), which acts as DNS resolver, over **UDP port 53**.
- Record types: **A** (IPv4 address), **AAAA** (IPv6 address), **HTTPS** (connection info for the site).
- Each query and its response share the same transaction ID (e.g. `0x9a51`).
- `gdmf.apple.com` is an alias (CNAME) of `gdmf.v.aaplimg.com`: one name can point to another.

### ICMP / ping (filter `icmp`)
- Echo request (`ttl=64`, macOS default) -> Echo reply from 8.8.8.8 (`ttl=116`).
- Google sends with TTL 128 -> 128 - 116 = **12 routers** between me and Google.
- Same `id` for the whole ping session, `seq` increases by 1 for each packet. Round trip ≈ 10 ms.

### TCP 3-way handshake (filter `tcp.flags.syn == 1`)
1. **SYN**: my Mac (random port, e.g. 61054) -> server port **443** (HTTPS): "I want to connect"
2. **SYN-ACK**: server -> my Mac: "OK, ready"
3. **ACK**: my Mac -> server: "Connection established" (not shown by this filter; use `tcp.stream eq N` to see the whole conversation)
- Most connections use **IPv6** (Ethernet type `0x86dd`), one uses IPv4 (`192.168.1.8 -> 17.171.47.23`, an Apple server).
- The destination MAC is always the Freebox: the server is on another network, so the frame goes to the **default gateway** (layer 2), which routes it (layer 3).

### Pourquoi c'est important en sécu
- **DNS is not encrypted** by default: anyone on the network can see which sites I visit. Attacks: DNS spoofing (fake answers), DNS tunneling (hiding data in DNS queries).
- **ICMP**: ping sweeps reveal live hosts; TTL values can hint at the OS (64 = Linux/macOS, 128 = Windows).
- **TCP handshake** = basis of port scanning: Nmap's SYN scan sends a SYN; a SYN-ACK back means the port is open. A SYN flood sends thousands of SYNs without finishing the handshake (denial of service).
- **IPv6** gives each device a public address: the router's firewall is what protects internal machines, not NAT.
- Wireshark is a key tool for SOC analysts: reading packets layer by layer to investigate incidents.

## Professor Messer - Common Ports (Network+ N10-009, 1.4)

| Port | Protocol | Use | Encrypted? |
| --- | --- | --- | --- |
| tcp/20, tcp/21 | **FTP** | File transfer (20 = data in active mode, 21 = control), username + password | No (password in clear) |
| tcp/22 | **SSH** | Encrypted remote terminal | Yes |
| tcp/22 | **SFTP** | File transfer over SSH | Yes |
| tcp/23 | **Telnet** | Remote terminal, like SSH but in clear | No |
| tcp/25 | **SMTP** | Email sending, server to server | No by default |
| tcp/587 | **SMTP** (submission) | Email sending from a client to its mail server, with authentication + TLS | Yes |
| udp/53 (tcp/53) | **DNS** | Name -> IP address; TCP for large transfers (zone transfers) | No by default |
| udp/67, udp/68 | **DHCP** | Automatic IP configuration (67 = server, 68 = client) | No |
| udp/69 | **TFTP** | Very simple file transfer (configs, firmware), no authentication | No |
| tcp/80 | **HTTP** | Web, in clear | No |
| tcp/443 | **HTTPS** | Web encrypted with TLS | Yes |
| udp/123 | **NTP** | Synchronizes the clocks of all devices with a time server | No |
| udp/161 | **SNMP** | Queries network devices for statistics | v1/v2 no, v3 yes |
| udp/162 | **SNMP traps** | Devices send alerts to the management server | v1/v2 no, v3 yes |
| tcp/389 | **LDAP** | Read/write in a network directory (users, groups) | No |
| tcp/636 | **LDAPS** | LDAP over TLS | Yes |
| tcp/445 | **SMB** (CIFS) | Windows file and printer sharing | Depends on version |
| udp/514 | **Syslog** | Sending logs to a central log server | No |
| tcp/1433 | **Microsoft SQL Server** | Database queries | Optional |
| tcp/3389 | **RDP** | Remote Windows desktop | Yes |
| tcp/5060, tcp/5061 | **SIP** | Setting up and managing VoIP calls (5060 in clear, 5061 with TLS) | 5061 only |

### Notes
- To receive email, clients use **IMAP** or **POP3** (SMTP is only for sending).
- DHCP: at home the box does it; in companies a dedicated DHCP server. A **DHCP reservation** always gives the same IP to the same MAC address.
- NTP matters because logs from different machines must have the same time to be compared during an investigation.

### Pourquoi c'est important en sécu
- Insecure protocol -> secure replacement: FTP -> **SFTP**, Telnet -> **SSH**, HTTP -> **HTTPS**, LDAP -> **LDAPS**, SNMP v1/v2 -> **SNMPv3**. Protocols in clear expose passwords to anyone sniffing the network (Wireshark).
- An Nmap scan lists the open ports of a machine: knowing the ports tells you which services run and what to attack or protect.
- **SMB (445)** and **RDP (3389)** exposed to the Internet are among the most attacked services (e.g. WannaCry ransomware used an SMB flaw in 2017).
- **Syslog** and **NTP** are the basis of a SOC: centralized logs with synchronized timestamps.

## Lab - Following a TCP stream and listing open ports

### Full HTTP conversation (`curl http://example.com`, filter `tcp.stream eq 12`)
- Handshake: `[SYN]` -> `[SYN, ACK]` -> `[ACK]`, then the data, then the closing with `[FIN, ACK]` packets.
- Ports: my Mac used a random source port (**59889**) to reach the server on port **80** (HTTP). The server's `[FIN, ACK]` goes `80 -> 59889`.
- **Request** (client, in red in Wireshark): `GET / HTTP/1.1`, `Host: example.com`, `User-Agent: curl/8.12.1` (the tool I used), `Accept: */*`.
- **Response** (server, in blue): `HTTP/1.1 200 OK`, then headers, then the HTML of the page.
- Interesting headers: `Server: cloudflare` (the site is behind the Cloudflare CDN), `cf-cache-status: HIT` (page served from Cloudflare's cache), `Allow: GET, HEAD` (accepted methods), `alt-svc: h3=":443"` (the site also offers HTTP/3 on port 443).

### Open ports on my Mac (`lsof -i -P | grep LISTEN`)

| Program | Ports | What it is | Listening on |
| --- | --- | --- | --- |
| rapportd | 59418, 58586, 58587 | Apple service that lets my Apple devices talk to each other (Handoff, Continuity) | `*` = all interfaces |
| ControlCenter | 5000, 7000 | **AirPlay receiver** (receiving video/music from another Apple device) | `*` = all interfaces |

- `*` = reachable from any device on the **local network**. `localhost` / `127.0.0.1` = reachable only from the Mac itself.
- From the Internet, the Freebox firewall normally blocks incoming connections.

### Pourquoi c'est important en sécu
- **HTTP is in clear**: anyone sniffing the network can read the full request and response (URLs, cookies, form data, passwords). That's why HTTPS is mandatory.
- Headers leak information useful for **reconnaissance**: `User-Agent` reveals the client tool, `Server` reveals the server/CDN technology.
- Every open port = a potential entry point (**attack surface**). Good practice: close unused services (e.g. disable the AirPlay receiver if not used) and enable the macOS firewall.
- `lsof` / `netstat` on a machine and **Nmap** from the network show the same thing from two sides: what's listening.

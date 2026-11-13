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

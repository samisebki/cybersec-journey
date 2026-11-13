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

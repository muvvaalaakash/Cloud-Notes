IP Address Classification for DevOps Engineers
What is an IP Address?
An IP address is a unique identifier assigned to every device on a network.

text

Format: 32 bits (IPv4) = 4 octets
Example: 192.168.1.100
         |   |  | |
         8   8  8 8 bits = 32 bits total
Traditional IP Address Classes
IP addresses are divided into 5 classes (A, B, C, D, E)

CLASS A
Range: 1.0.0.0 to 126.255.255.255
text

Format:  Network . Host . Host . Host
Bits:    8 bits  . 24 bits
Prefix:  /8
Example:
text

IP Address:  10.0.0.1
Network:     10.0.0.0
Host Range:  10.0.0.1 - 10.255.255.254
Broadcast:   10.255.255.255

Total Hosts: 2^24 - 2 = 16,777,214 hosts
DevOps Use Case:
text

✅ Large cloud provider networks (AWS, Azure, GCP)
✅ Large enterprise data centers
✅ AWS VPC default CIDR → 10.0.0.0/8
CLASS B
Range: 128.0.0.0 to 191.255.255.255
text

Format:  Network . Network . Host . Host
Bits:    16 bits           . 16 bits
Prefix:  /16
Example:
text

IP Address:  172.16.0.1
Network:     172.16.0.0
Host Range:  172.16.0.1 - 172.16.255.254
Broadcast:   172.16.255.255

Total Hosts: 2^16 - 2 = 65,534 hosts
DevOps Use Case:
text

✅ Medium to large company networks
✅ Kubernetes cluster networking
✅ Docker default subnet → 172.17.0.0/16
✅ AWS VPC → 172.16.0.0/16
CLASS C
Range: 192.0.0.0 to 223.255.255.255
text

Format:  Network . Network . Network . Host
Bits:    24 bits                     . 8 bits
Prefix:  /24
Example:
text

IP Address:  192.168.1.1
Network:     192.168.1.0
Host Range:  192.168.1.1 - 192.168.1.254
Broadcast:   192.168.1.255

Total Hosts: 2^8 - 2 = 254 hosts
DevOps Use Case:
text

✅ Small office networks
✅ Local development environments
✅ Home lab setups
✅ Small microservice subnets in cloud
CLASS D (Multicast)
Range: 224.0.0.0 to 239.255.255.255
text

Used for: Multicast groups (one-to-many communication)
Not assigned to individual hosts
DevOps Use Case:
text

✅ Kubernetes uses multicast for service discovery
✅ Network protocols like OSPF, EIGRP
✅ Video streaming servers
✅ Distributed system communication
CLASS E (Reserved/Experimental)
Range: 240.0.0.0 to 255.255.255.255
text

Used for: Research and experimental purposes
Not used in production
Quick Summary Table
Class	Range Start	Range End	Prefix	Total Hosts	Use Case
A	1.0.0.0	126.255.255.255	/8	16,777,214	Large networks
B	128.0.0.0	191.255.255.255	/16	65,534	Medium networks
C	192.0.0.0	223.255.255.255	/24	254	Small networks
D	224.0.0.0	239.255.255.255	N/A	Multicast only	Streaming, Discovery
E	240.0.0.0	255.255.255.255	N/A	Experimental	Research
Private IP Address Ranges
These are used inside networks (not routable on internet)

text

Class A:  10.0.0.0    - 10.255.255.255    (10.0.0.0/8)
Class B:  172.16.0.0  - 172.31.255.255    (172.16.0.0/12)
Class C:  192.168.0.0 - 192.168.255.255   (192.168.0.0/16)
Special IP Addresses
text

127.0.0.1       → Loopback (localhost) - test your own machine
0.0.0.0         → Default route / All networks
255.255.255.255 → Broadcast to all devices
169.254.x.x     → APIPA (Auto assigned when DHCP fails)
Real-World DevOps Scenarios
Scenario 1: AWS VPC Setup
text

VPC CIDR:          10.0.0.0/16       ← Class A Private
Public Subnet:     10.0.1.0/24       ← 254 hosts (web servers)
Private Subnet:    10.0.2.0/24       ← 254 hosts (app servers)
Database Subnet:   10.0.3.0/24       ← 254 hosts (RDS)
Scenario 2: Kubernetes Cluster
text

Node Network:      172.16.0.0/16     ← Class B Private
Pod Network:       10.244.0.0/16     ← Class A Private (Flannel)
Service Network:   10.96.0.0/12      ← Cluster IP range
Scenario 3: Docker Networking
text

Default Bridge:    172.17.0.0/16     ← Class B
Container 1:       172.17.0.2
Container 2:       172.17.0.3
Host:              172.17.0.1
How to Check IP Class (Quick Trick)
Look at the first octet:

text

1   - 126   → Class A
128 - 191   → Class B
192 - 223   → Class C
224 - 239   → Class D
240 - 255   → Class E
Example:
text

10.0.0.1    → First octet = 10  → Class A ✅
172.16.0.1  → First octet = 172 → Class B ✅
192.168.1.1 → First octet = 192 → Class C ✅
224.0.0.1   → First octet = 224 → Class D ✅
Key Takeaways for DevOps Engineers
text

✅ Class A → Large infra (AWS VPC, enterprise DC)
✅ Class B → Medium infra (Kubernetes, Docker)
✅ Class C → Small subnets (microservices, labs)
✅ Always use Private IPs inside your infrastructure
✅ Public IPs only for internet-facing resources
✅ CIDR helps you slice these classes efficiently



CIDR means:

Classless Inter-Domain Routing

It defines:

network boundary
number of hosts
routing scope

Example:

192.168.1.0/24

This means:

network = 192.168.1.x
/24 defines network size
3. CIDR Is Basically a Divider

CIDR divides IP into:

network portion
host portion

Example:

192.168.1.10/24

Means:

Part	Value
Network	192.168.1
Host	10

All devices with:

192.168.1.x

belong to same network.

4. What Does "/24" Mean?

IPv4 has:

32 bits total

/24 means:

24 bits are network bits

Remaining:

8 bits are host bits

1. Network Overview Section

Start with high-level architecture.

Example
Project Name: Shareify Platform

Cloud Provider: AWS
Region: ap-south-1

VPC CIDR: 10.0.0.0/16

Purpose:
Main network for hosting Kubernetes cluster, databases,
load balancers, monitoring stack, and internal services.
2. CIDR Allocation Table

MOST IMPORTANT SECTION.

Example
Component	CIDR	Purpose
VPC	10.0.0.0/16	Main cloud network
Public Subnet	10.0.1.0/24	Internet-facing resources
Private App Subnet	10.0.2.0/24	Backend microservices
Database Subnet	10.0.3.0/24	PostgreSQL databases
Kubernetes Pod CIDR	10.244.0.0/16	Pod networking
Kubernetes Service CIDR	10.96.0.0/12	ClusterIP services
VPN CIDR	192.168.1.0/24	Office VPN network
3. Subnet Explanation

Document WHY subnet exists.

Example
Public Subnet (10.0.1.0/24)

Purpose:
Hosts internet-facing components such as:
- Application Load Balancer
- NAT Gateway
- Bastion Host

Routing:
Connected to Internet Gateway.

Security:
Only required ports exposed publicly.
4. Private Subnet Documentation
Example
Private Application Subnet (10.0.2.0/24)

Purpose:
Hosts internal backend services:
- API Gateway
- User Service
- Booking Service
- Inventory Service

Routing:
No direct internet access.
Outbound traffic routed through NAT Gateway.

Security:
Accessible only through internal load balancing and Kubernetes services.
5. Database Subnet Documentation
Example
Database Subnet (10.0.3.0/24)

Purpose:
Hosts PostgreSQL StatefulSets and persistent database services.

Security:
- No public internet access
- Accessible only from backend services
- Restricted using security groups and network policies

High Availability:
Distributed across multiple availability zones.
6. Kubernetes Networking Documentation

Critical for DevOps documentation.

Example
Kubernetes Networking

Pod CIDR: 10.244.0.0/16
Service CIDR: 10.96.0.0/12

Explanation:
- Each pod receives unique IP from Pod CIDR
- Services receive virtual ClusterIP from Service CIDR
- CIDRs are isolated to prevent overlap with VPC network

CNI Plugin:
Calico

DNS:
CoreDNS used for internal service discovery.
7. Routing Documentation

Most beginners skip this. Big mistake.

Example
Destination	Target
10.0.0.0/16	Local VPC
0.0.0.0/0	Internet Gateway
Private Subnets	NAT Gateway
8. Security Documentation

Document allowed CIDRs.

Example
Security Rules

SSH Access:
Allowed only from office CIDR:
192.168.1.0/24

Public Access:
HTTP/HTTPS open to:
0.0.0.0/0

Database Access:
Restricted to backend subnet CIDRs only.
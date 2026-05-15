https://www.cloudthat.com/resources/blog/site-to-site-vpn-connection-between-aws-azure

approach we followed 

components involved

Azure to AWS Site-to-Site VPN Connection - Complete Explanation
Overview
You have established a Site-to-Site VPN (IPsec/IKE) tunnel between Microsoft Azure and Amazon Web Services (AWS), creating a secure, encrypted connection between two cloud environments over the public internet.

Resources Used and Their Roles
Azure Side Resources
1. Resource Group
What it is: A logical container that holds all related Azure resources
Role: Organizes and manages all the resources (VNet, Gateway, Connections) created for this VPN setup in one place
Why needed: Azure requires every resource to belong to a Resource Group for management, billing, and access control
2. Azure Virtual Network (VNet) — CIDR: 172.10.0.0/16
What it is: A private, isolated network in Azure cloud — the fundamental building block for your private network
Role: Provides the address space where Azure VMs and services reside
Analogy: Think of it as your private data center network in Azure
Contains 65,536 IP addresses (172.10.0.0 to 172.10.255.255)
3. Subnet — CIDR: 172.10.1.0/24
What it is: A subdivision of the VNet where you deploy actual resources (VMs, databases, etc.)
Role: Segments the VNet into smaller networks for better organization and security
Contains 256 IP addresses (172.10.1.0 to 172.10.1.255)
This is where your Azure VM is deployed
4. Gateway Subnet — CIDR: 172.10.5.0/27
What it is: A special, dedicated subnet exclusively reserved for the Virtual Network Gateway
Role: Hosts the VPN gateway instances (Azure deploys two hidden VMs in this subnet that handle encryption/decryption and routing)
Contains 32 IP addresses (172.10.5.0 to 172.10.5.31)
Why /27: Microsoft recommends at least /27 for gateway subnets to accommodate future scaling
Critical Rule: No other resources (VMs, etc.) can be deployed in this subnet
5. Azure Virtual Network Gateway (VPN Gateway)
What it is: The most critical resource — a managed VPN appliance that Azure deploys in the Gateway Subnet
Role:
Acts as the VPN endpoint on the Azure side
Performs IPsec/IKE negotiation with AWS
Encrypts outgoing traffic destined for AWS
Decrypts incoming traffic from AWS
Handles tunnel establishment and maintenance
Gets a Public IP address — this IP is given to AWS as the "Customer Gateway" IP
Internally: Azure deploys two VM instances in active-standby mode for high availability
6. Local Network Gateway (2 instances created)
What it is: A representation/reference of the remote (AWS) VPN endpoint in Azure
Role:
Tells Azure "where AWS is" — stores the AWS VPN tunnel public IP addresses
Stores the AWS VPC CIDR range (192.16.0.0/16) so Azure knows what traffic to route through the tunnel
Acts as the logical definition of the remote site
Why 2 instances: AWS provides two VPN tunnels for high availability, so each Local Network Gateway represents one AWS tunnel endpoint
Information stored:
AWS Tunnel 1 Public IP → Local Network Gateway 1
AWS Tunnel 2 Public IP → Local Network Gateway 2
Address Space: 192.16.0.0/16 (AWS VPC range)
7. Connection (2 connections created)
What it is: The logical link between the Azure Virtual Network Gateway and the Local Network Gateway
Role:
Binds the Azure VPN Gateway to each AWS tunnel endpoint
Contains the Pre-Shared Key (PSK) — the secret password both sides use for IPsec authentication
Defines the connection type as Site-to-Site (IPsec)
The shared key comes from the AWS VPN configuration file
8. Azure Route Table
What it is: A routing table associated with the Azure subnet (172.10.1.0/24)
Role: Contains routing rules that tell Azure where to send traffic
Key Route Added:
text

Destination: 192.16.0.0/16 (AWS VPC) → Next Hop: Virtual Network Gateway
Meaning: "Any traffic destined for the AWS network should be forwarded to the VPN Gateway for encryption and tunneling"
9. Network Security Group (NSG)
What it is: A virtual firewall for controlling inbound and outbound traffic
Role: Applied to the Azure subnet to allow traffic from/to the AWS VPC CIDR range (192.16.0.0/16)
Rule Added: Allow inbound/outbound traffic to 192.16.0.0/16
10. Azure Virtual Machine
What it is: A compute instance deployed in the Azure subnet (172.10.1.0/24)
Role: Used to test connectivity by RDP-ing to the AWS EC2 instance through the VPN tunnel
AWS Side Resources
1. AWS Virtual Private Cloud (VPC) — CIDR: 192.16.0.0/16
What it is: AWS's equivalent of Azure VNet — a private, isolated network in AWS
Role: Provides the address space where AWS EC2 instances and services reside
Contains 65,536 IP addresses (192.16.0.0 to 192.16.255.255)
2. AWS Subnet — CIDR: 192.16.1.0/24
What it is: A subdivision of the VPC
Role: Where the EC2 instance is deployed
Associated with a Route Table for proper routing
3. Customer Gateway
What it is: A representation of the remote (Azure) VPN endpoint in AWS
Role:
Tells AWS "where Azure is"
Stores the Azure Virtual Network Gateway's Public IP
AWS equivalent of Azure's Local Network Gateway (but in reverse direction)
IP Address configured: The Public IP of the Azure Virtual Network Gateway
4. AWS Virtual Private Gateway (VGW)
What it is: AWS's managed VPN endpoint — equivalent to Azure's Virtual Network Gateway
Role:
Acts as the VPN endpoint on the AWS side
Performs IPsec/IKE negotiation with Azure
Encrypts outgoing traffic destined for Azure
Decrypts incoming traffic from Azure
Attached to the VPC to route traffic
Provides two tunnel endpoints with two different public IPs for high availability
5. Site-to-Site VPN Connection
What it is: The actual VPN connection configuration in AWS
Role:
Links the Customer Gateway (Azure endpoint) with the Virtual Private Gateway (AWS endpoint)
Defines static routing with Azure VNet CIDR (172.10.0.0/16)
Creates two IPsec tunnels automatically for redundancy
Generates the configuration file containing:
Two tunnel public IPs
Two pre-shared keys
IPsec/IKE parameters
6. AWS Route Table
What it is: Routing table associated with the AWS subnet
Key Route Added:
text

Destination: 172.10.0.0/16 (Azure VNet) → Target: Virtual Private Gateway
Meaning: "Any traffic destined for the Azure network should be forwarded to the Virtual Private Gateway for encryption and tunneling"
7. Security Group
What it is: AWS's virtual firewall (stateful) attached to the EC2 instance
Role: Allows inbound traffic from Azure VNet CIDR range (172.10.0.0/16)
Rule Added: Allow RDP (port 3389) from 172.10.0.0/16
8. EC2 Instance (Windows)
What it is: A compute instance deployed in the AWS subnet — without a public IP
Role: Used to test connectivity — reachable only through the VPN tunnel via private IP
No public IP is critical: This proves the connection works through the VPN tunnel, not over the public internet
How Communication Happens
Azure VM → AWS EC2 (Azure to AWS)
Here is the step-by-step packet flow:

text

┌─────────────────────────────────────────────────────────────────────────────┐
│                        AZURE TO AWS PACKET FLOW                             │
└─────────────────────────────────────────────────────────────────────────────┘

Step 1: Azure VM (172.10.1.x) initiates RDP connection to AWS EC2 (192.16.1.x)
                    │
                    ▼
Step 2: Packet hits the Azure Route Table
        Route Table lookup: 
        "Destination 192.16.0.0/16 → Next Hop: Virtual Network Gateway"
                    │
                    ▼
Step 3: Packet is forwarded to Azure Virtual Network Gateway
        The gateway identifies this traffic belongs to the VPN tunnel
                    │
                    ▼
Step 4: IPsec Encryption
        ┌──────────────────────────────────────────┐
        │  Original Packet:                         │
        │  Src: 172.10.1.x  Dst: 192.16.1.x       │
        │                                           │
        │  Gets ENCRYPTED using:                    │
        │  - Pre-Shared Key (from config file)      │
        │  - IKE/IPsec protocols                    │
        │  - ESP (Encapsulating Security Payload)   │
        │                                           │
        │  Encapsulated in NEW packet:              │
        │  Src: Azure Gateway Public IP             │
        │  Dst: AWS Tunnel Public IP                │
        │  Payload: [Encrypted Original Packet]     │
        └──────────────────────────────────────────┘
                    │
                    ▼
Step 5: Encrypted packet travels over the PUBLIC INTERNET
        (Through Azure's backbone → Internet → AWS backbone)
                    │
                    ▼
Step 6: Packet arrives at AWS Virtual Private Gateway
        ┌──────────────────────────────────────────┐
        │  IPsec Decryption:                        │
        │  - Validates Pre-Shared Key               │
        │  - Decrypts ESP payload                   │
        │  - Extracts original packet:              │
        │    Src: 172.10.1.x  Dst: 192.16.1.x     │
        └──────────────────────────────────────────┘
                    │
                    ▼
Step 7: AWS Route Table routes the decrypted packet
        to the subnet 192.16.1.0/24
                    │
                    ▼
Step 8: Security Group check — allows traffic from 172.10.0.0/16
                    │
                    ▼
Step 9: Packet arrives at AWS EC2 instance (192.16.1.x)
        RDP session established!
AWS EC2 → Azure VM (AWS to Azure — Return Path)
text

┌─────────────────────────────────────────────────────────────────────────────┐
│                        AWS TO AZURE PACKET FLOW                             │
└─────────────────────────────────────────────────────────────────────────────┘

Step 1: AWS EC2 (192.16.1.x) sends response to Azure VM (172.10.1.x)
                    │
                    ▼
Step 2: AWS Route Table lookup:
        "Destination 172.10.0.0/16 → Target: Virtual Private Gateway"
                    │
                    ▼
Step 3: Packet forwarded to AWS Virtual Private Gateway
                    │
                    ▼
Step 4: IPsec Encryption at AWS side
        ┌──────────────────────────────────────────┐
        │  Original Packet:                         │
        │  Src: 192.16.1.x  Dst: 172.10.1.x       │
        │                                           │
        │  Gets ENCRYPTED                           │
        │                                           │
        │  Encapsulated in NEW packet:              │
        │  Src: AWS Tunnel Public IP                │
        │  Dst: Azure Gateway Public IP             │
        │  Payload: [Encrypted Original Packet]     │
        └──────────────────────────────────────────┘
                    │
                    ▼
Step 5: Encrypted packet travels over PUBLIC INTERNET
                    │
                    ▼
Step 6: Arrives at Azure Virtual Network Gateway
        IPsec Decryption → Extracts original packet
                    │
                    ▼
Step 7: Azure NSG allows traffic from 192.16.0.0/16
                    │
                    ▼
Step 8: Packet arrives at Azure VM (172.10.1.x)
Architecture Diagram
text

                    AZURE CLOUD                                         AWS CLOUD
    ┌──────────────────────────────────┐               ┌──────────────────────────────────┐
    │   Resource Group                  │               │   AWS Account                     │
    │  ┌────────────────────────────┐   │               │  ┌────────────────────────────┐   │
    │  │  VNet: 172.10.0.0/16      │   │               │  │  VPC: 192.16.0.0/16       │   │
    │  │                            │   │               │  │                            │   │
    │  │  ┌──────────────────────┐  │   │               │  │  ┌──────────────────────┐  │   │
    │  │  │ Subnet: 172.10.1.0/24│  │   │               │  │  │ Subnet: 192.16.1.0/24│  │   │
    │  │  │                      │  │   │               │  │  │                      │  │   │
    │  │  │   ┌──────────┐       │  │   │               │  │  │   ┌──────────┐       │  │   │
    │  │  │   │ Azure VM │       │  │   │               │  │  │   │ EC2 Inst │       │  │   │
    │  │  │   │172.10.1.x│       │  │   │               │  │  │   │192.16.1.x│       │  │   │
    │  │  │   └────┬─────┘       │  │   │               │  │  │   └────┬─────┘       │  │   │
    │  │  │   [NSG]│[Route Table]│  │   │               │  │  │   [SG] │[Route Table]│  │   │
    │  │  └────────┼─────────────┘  │   │               │  │  └────────┼─────────────┘  │   │
    │  │           │                │   │               │  │           │                │   │
    │  │  ┌────────┼─────────────┐  │   │               │  │           │                │   │
    │  │  │GatewaySubnet:        │  │   │               │  │           │                │   │
    │  │  │  172.10.5.0/27       │  │   │               │  │           │                │   │
    │  │  │  ┌───────────────┐   │  │   │               │  │  ┌───────┴───────┐        │   │
    │  │  │  │  VPN Gateway  │   │  │   │               │  │  │Virtual Private│        │   │
    │  │  │  │  (Public IP)  │   │  │   │               │  │  │   Gateway     │        │   │
    │  │  │  └───────┬───────┘   │  │   │               │  │  │  (2 Tunnels)  │        │   │
    │  │  └──────────┼───────────┘  │   │               │  │  └───────┬───────┘        │   │
    │  └─────────────┼──────────────┘   │               │  └──────────┼────────────────┘   │
    │                │                   │               │             │                     │
    │  ┌─────────────┼───────────────┐   │               │  ┌──────────┼──────────────┐     │
    │  │Local Network│Gateway 1      │   │               │  │Customer  │Gateway       │     │
    │  │(AWS Tunnel 1│IP + PSK)      │   │               │  │(Azure GW │Public IP)    │     │
    │  │Local Network│Gateway 2      │   │               │  └──────────┼──────────────┘     │
    │  │(AWS Tunnel 2│IP + PSK)      │   │               │             │                     │
    │  └─────────────┼───────────────┘   │               │             │                     │
    └────────────────┼───────────────────┘               └─────────────┼─────────────────────┘
                     │                                                 │
                     │         ┌───────────────────────┐               │
                     └─────────┤   PUBLIC INTERNET      ├──────────────┘
                               │                       │
                               │  IPsec/IKE Encrypted  │
                               │  Tunnel 1 ═══════════ │
                               │  Tunnel 2 ═══════════ │
                               └───────────────────────┘
Key Concepts Explained
IPsec/IKE Protocol
IKE (Internet Key Exchange): Negotiates the security parameters between Azure and AWS — agrees on encryption algorithms, authentication methods, and establishes the security association
IPsec: The actual protocol that encrypts and authenticates the data packets
ESP (Encapsulating Security Payload): Provides confidentiality by encrypting the original packet
Why Two Tunnels?
AWS always provides two tunnels for high availability:

Tunnel 1: Primary path
Tunnel 2: Failover path
If one tunnel goes down, traffic automatically fails over to the second tunnel
Each tunnel terminates on a different AWS endpoint in different availability infrastructure
Static Routing
In this setup, you chose static routing (as opposed to BGP dynamic routing)
You manually specified which CIDR ranges should go through the tunnel:
Azure side: Route 192.16.0.0/16 → VPN Gateway
AWS side: Route 172.10.0.0/16 → Virtual Private Gateway
Pre-Shared Key (PSK)
A shared secret configured on both sides
Used during IKE negotiation to authenticate the two endpoints
Found in the AWS VPN configuration file downloaded in Step 6
Resource Mapping (Azure ↔ AWS Equivalents)
Function	Azure Resource	AWS Resource
Private Network	Virtual Network (VNet)	Virtual Private Cloud (VPC)
Network Segment	Subnet	Subnet
VPN Endpoint (Local)	Virtual Network Gateway	Virtual Private Gateway
VPN Endpoint (Remote)	Local Network Gateway	Customer Gateway
VPN Link	Connection	Site-to-Site VPN Connection
Routing	Route Table	Route Table
Firewall	Network Security Group	Security Group
Compute	Virtual Machine	EC2 Instance
Summary of Communication Flow
Traffic originates from a VM in one cloud (e.g., Azure VM at 172.10.1.x)
Route table identifies the destination (192.16.0.0/16) belongs to the remote cloud and forwards it to the VPN Gateway
VPN Gateway encrypts the packet using IPsec with the pre-shared key
Encrypted packet traverses the public internet between the two gateway public IPs
Remote VPN Gateway decrypts the packet and extracts the original packet
Remote route table delivers the decrypted packet to the destination VM
Security Group/NSG validates that the traffic is allowed
Packet arrives at the destination VM — communication is complete
The entire process happens in milliseconds, and from the VMs' perspective, they communicate as if they are on the same private network — they never see the encryption, tunneling, or internet traversal. The EC2 instance having no public IP proves that the communication is entirely through the private VPN tunnel.
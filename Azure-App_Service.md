## Platform Engineering

Platform engineering is the discipline of designing and building internal developer platforms (IDPs) — the toolchains, workflows, and self-service infrastructure that enable development teams to deliver software faster and more reliably.

The core idea: instead of every dev team reinventing DevOps from scratch, a dedicated **platform team** builds a "golden path" — opinionated, curated tooling that abstracts away infrastructure complexity.

**What it includes:**
- CI/CD pipelines and deployment automation
- Infrastructure provisioning (often via Terraform/Pulumi)
- Observability stacks (logging, metrics, tracing)
- Developer portals (e.g. Backstage)
- Security and compliance guardrails baked in

**Why it matters:** It shifts developers from *managing* infrastructure to *using* it — reducing cognitive load and improving delivery velocity.

---

## Platform as a Service (PaaS)

PaaS is a **cloud delivery model** where a provider manages the underlying infrastructure (servers, OS, networking, runtime) and you just bring your code and data.

| You manage | Provider manages |
|---|---|
| Application code | Servers & VMs |
| Data | Operating system |
| Configuration | Networking |
| | Runtime & middleware |

**Examples:** Google App Engine, Heroku, Railway, Render, and — most relevantly — **Azure App Service**.

PaaS sits between IaaS (you manage VMs) and SaaS (you manage nothing). The tradeoff is **simplicity vs. control**.

---

## Azure App Service

Azure App Service is Microsoft's fully managed PaaS for hosting web applications, REST APIs, and mobile backends.

**Key capabilities:**

**Supported runtimes** — .NET, Node.js, Python, Java, PHP, Ruby, and custom containers (Docker).

**Deployment options:**
- Git push, GitHub Actions, Azure DevOps pipelines
- ZIP deploy, FTP, or container registry pull

**Scaling:**
- *Vertical* — scale up to bigger SKUs (more CPU/RAM)
- *Horizontal* — scale out to multiple instances manually or via autoscale rules

**Built-in features:**
- Custom domains + managed TLS/SSL certificates
- Authentication/authorization via Entra ID, Google, Facebook, etc. (Easy Auth)
- Deployment slots — run staging and production side by side, then swap with zero downtime
- WebJobs — background task runner baked in
- VNet integration for private networking

**App Service Plans** define the compute tier — Free/Shared tiers for dev/test, Basic through Premium tiers for production, and Isolated (ASE) for fully private, dedicated environments.

---

## How They Relate

```
Platform Engineering  ──►  builds internal platforms using tools like...
                                │
                                ▼
                         Azure App Service (PaaS)
                                │
                         abstracts away VMs, OS,
                         networking so teams just
                         deploy code
```

Platform engineering is the *practice and discipline*; PaaS like Azure App Service is one of the *building blocks* platform teams use. A platform team might standardize on App Service as the approved deployment target, wrap it with their own CI/CD templates, and expose it to dev teams via a self-service portal — that's platform engineering in action.

Deploying Microservices App to Azure App Services with VNet & Cosmos DB
Architecture Overview
text

Internet
    │
    ▼
┌─────────────────────────────────────────────────────┐
│  Resource Group: rg-foodapp-prod                    │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  VNet: vnet-foodapp (10.0.0.0/16)           │   │
│  │                                             │   │
│  │  ┌──────────────────────────────────────┐   │   │
│  │  │ Subnet 1: snet-appservice            │   │   │
│  │  │ (10.0.1.0/24) - App Services         │   │   │
│  │  │ [Frontend, Gateway, All Services]    │   │   │
│  │  └──────────────────────────────────────┘   │   │
│  │                                             │   │
│  │  ┌──────────────────────────────────────┐   │   │
│  │  │ Subnet 2: snet-database              │   │   │
│  │  │ (10.0.2.0/24) - Private Endpoints    │   │   │
│  │  │ [Cosmos DB Private Endpoint]         │   │   │
│  │  └──────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Cosmos DB Account (Private Endpoint)               │
│  [users-db, restaurants-db, orders-db]              │
└─────────────────────────────────────────────────────┘
PHASE 1: Create Resource Group
Step 1.1 - Create Resource Group
text

Azure Portal → Search "Resource Groups" → Click "+ Create"
Field	Value
Subscription	Your subscription
Resource group	rg-foodapp-prod
Region	East US (pick closest to you)
text

Click "Review + Create" → Click "Create"
PHASE 2: Create Virtual Network with Two Subnets
Step 2.1 - Create VNet
text

Portal → Search "Virtual Networks" → Click "+ Create"
Basics Tab:

Field	Value
Resource Group	rg-foodapp-prod
Name	vnet-foodapp
Region	East US
IP Addresses Tab:

text

Default address space: 10.0.0.0/16

DELETE the default subnet first (click delete icon)

Then ADD two new subnets:
Step 2.2 - Add Subnet 1 (App Services)
text

Click "+ Add Subnet"
Field	Value
Name	snet-appservice
Starting address	10.0.1.0
Size	/24 (256 addresses)
text

⚠️ IMPORTANT SETTING:
Scroll down → Find "Delegate subnet to a service"
Select: Microsoft.Web/serverFarms
Click Add

Step 2.3 - Add Subnet 2 (Database/Private Endpoints)
text

Click "+ Add Subnet" again
Field	Value
Name	snet-database
Starting address	10.0.2.0
Size	/24 (256 addresses)
text

⚠️ Leave delegation as "None" for this subnet
Private endpoint network policies: DISABLED (default is fine)
Click Add

Security Tab: Leave defaults

text

Click "Review + Create" → Click "Create"
Wait for deployment to complete ✅
PHASE 3: Create Cosmos DB Account
Step 3.1 - Start Cosmos DB Creation
text

Portal → Search "Azure Cosmos DB" → Click "+ Create"
Select API:

text

Choose: "Azure Cosmos DB for MongoDB"
         (since your app uses MongoDB)
Click "Create"
Basics Tab:

Field	Value
Resource Group	rg-foodapp-prod
Account Name	cosmos-foodapp-prod (must be globally unique)
Location	East US
Capacity mode	Serverless (cost effective for start)
Version	4.2
Step 3.2 - Configure Networking (Critical Step)
text

Click "Next: Global Distribution" → Skip (leave defaults)
Click "Next: Networking"
Networking Tab:

Field	Value
Connectivity method	Private endpoint ← SELECT THIS
Allow Azure services	Yes
text

Click "+ Add" under Private Endpoint section
Create Private Endpoint popup:

Field	Value
Name	pe-cosmos-foodapp
Region	East US
Target sub-resource	MongoDB
Virtual Network	vnet-foodapp
Subnet	snet-database ← database subnet
Integrate with DNS	Yes
Private DNS Zone	(auto-created) privatelink.mongo.cosmos.azure.com
Click OK

Step 3.3 - Backup & Review
text

Click "Next: Backup Policy" → Leave defaults
Click "Next: Encryption" → Leave defaults
Click "Review + Create" → Click "Create"

⏳ Wait 5-10 minutes for Cosmos DB deployment
PHASE 4: Create Databases in Cosmos DB
Step 4.1 - Create Three Databases
text

Portal → Go to your Cosmos DB account "cosmos-foodapp-prod"
Left menu → Click "Data Explorer"
Create users-db:

text

Click "New Database"
Field	Value
Database id	users-db
Provision throughput	Unchecked (serverless)
Click OK

text

Repeat for:
- "restaurants-db"  
- "orders-db"
Step 4.2 - Create Collections in Each Database
In users-db:

text

Click the "..." next to users-db → "New Collection"
Field	Value
Database id	users-db (existing)
Collection id	users
Shard key	_id
In restaurants-db:

text

Collection id: "restaurants" | Shard key: "_id"
In orders-db:

text

Collection id: "orders" | Shard key: "_id"
Step 4.3 - Get Connection Strings
text

Cosmos DB Account → Left Menu → "Connection strings"

Copy these values (you'll need them later):
┌─────────────────────────────────────────────┐
│ PRIMARY CONNECTION STRING → COPY & SAVE     │
│ mongodb://cosmos-foodapp-prod:xxxxx@...      │
└─────────────────────────────────────────────┘

Create individual connection strings per database:
- Add "?dbName=users-db" at end for user service
- Add "?dbName=restaurants-db" for restaurant service  
- Add "?dbName=orders-db" for order service
PHASE 5: Create App Service Plan
Step 5.1 - Create Shared App Service Plan
text

Portal → Search "App Service Plans" → Click "+ Create"
Field	Value
Resource Group	rg-foodapp-prod
Name	asp-foodapp-prod
Operating System	Linux
Region	East US
Pricing Tier	Click "Explore pricing tiers"
Pricing Tier Selection:

text

⚠️ IMPORTANT: Must be P1v2 or higher for VNet Integration
Select: "P1 v2" (Production → P1 v2)
This supports VNet Integration feature
Click Review + Create → Create

PHASE 6: Create All App Services (5 Services)
You will create 5 App Services using the same plan:

app-frontend (React - Port 3000)
app-gateway (API Gateway - Port 5000)
app-user-service (Port 5001)
app-restaurant-service (Port 5002)
app-order-service (Port 5003)
Step 6.1 - Create Each App Service
text

Portal → Search "App Services" → Click "+ Create" → "Web App"
Repeat this for ALL 5 services:

Basics Tab:

Field	Service 1	Service 2	Service 3	Service 4	Service 5
Resource Group	rg-foodapp-prod	same	same	same	same
Name	app-foodapp-frontend	app-foodapp-gateway	app-foodapp-user	app-foodapp-restaurant	app-foodapp-order
Runtime	Node 18 LTS	Node 18 LTS	Node 18 LTS	Node 18 LTS	Node 18 LTS
OS	Linux	Linux	Linux	Linux	Linux
Region	East US	East US	East US	East US	East US
App Service Plan	asp-foodapp-prod	same	same	same	same
text

Click "Review + Create" → "Create" for each one
PHASE 7: Configure VNet Integration for All App Services
Step 7.1 - Enable VNet Integration
text

⚠️ Do this for ALL 5 App Services
text

Go to App Service → (e.g., app-foodapp-gateway)
Left Menu → "Networking"
Click "VNet Integration" → "Add VNet"
Field	Value
Virtual Network	vnet-foodapp
Subnet	snet-appservice ← App service subnet
Click Connect

text

✅ Repeat for all 5 App Services:
- app-foodapp-frontend
- app-foodapp-gateway
- app-foodapp-user
- app-foodapp-restaurant
- app-foodapp-order
Step 7.2 - Enable Route All Traffic Through VNet
text

For EACH App Service after VNet Integration:
Networking → VNet Integration → Click on your VNet
Toggle: "Route All" → ON
Click Save
PHASE 8: Configure Environment Variables (App Settings)
Step 8.1 - Gateway Service Settings
text

app-foodapp-gateway → Configuration → Application Settings
Click "+ New application setting" for each:
Setting Name	Value
PORT	5000
USER_SERVICE_URL	https://app-foodapp-user.azurewebsites.net
RESTAURANT_SERVICE_URL	https://app-foodapp-restaurant.azurewebsites.net
ORDER_SERVICE_URL	https://app-foodapp-order.azurewebsites.net
NODE_ENV	production
Click Save

Step 8.2 - User Service Settings
text

app-foodapp-user → Configuration → Application Settings
Setting Name	Value
PORT	5001
MONGODB_URI	<cosmos_connection_string>?dbName=users-db
NODE_ENV	production
Step 8.3 - Restaurant Service Settings
text

app-foodapp-restaurant → Configuration → Application Settings
Setting Name	Value
PORT	5002
MONGODB_URI	<cosmos_connection_string>?dbName=restaurants-db
NODE_ENV	production
Step 8.4 - Order Service Settings
text

app-foodapp-order → Configuration → Application Settings
Setting Name	Value
PORT	5003
MONGODB_URI	<cosmos_connection_string>?dbName=orders-db
NODE_ENV	production
Step 8.5 - Frontend Settings
text

app-foodapp-frontend → Configuration → Application Settings
Setting Name	Value
REACT_APP_API_URL	https://app-foodapp-gateway.azurewebsites.net
NODE_ENV	production
PHASE 9: Configure Private DNS Zone for Cosmos DB
Step 9.1 - Verify Private DNS Zone Exists
text

Portal → Search "Private DNS Zones"
You should see: "privatelink.mongo.cosmos.azure.com"
(Created automatically when you created private endpoint)
Step 9.2 - Link DNS Zone to VNet
text

Click on "privatelink.mongo.cosmos.azure.com"
Left Menu → "Virtual network links"
Click "+ Add"
Field	Value
Link name	link-vnet-foodapp
Virtual network	vnet-foodapp
Enable auto registration	No
Click OK

text

✅ This ensures App Services can resolve Cosmos DB 
   private endpoint DNS name
PHASE 10: Deploy Your Application Code
Step 10.1 - Deploy via GitHub Actions (Recommended)
text

Each App Service → "Deployment Center"
Source: GitHub
Organization: your-org
Repository: your-repo
Branch: main
Configure build settings for each service:

text

For each service, set the correct subfolder:

app-foodapp-frontend  → App location: "/frontend"
app-foodapp-gateway   → App location: "/api-gateway"
app-foodapp-user      → App location: "/user-service"
app-foodapp-restaurant → App location: "/restaurant-service"
app-foodapp-order     → App location: "/order-service"
Step 10.2 - Alternative: Deploy via ZIP
text

App Service → "Advanced Tools" → Go (Kudu)
Debug console → CMD
Drag and drop your ZIP file

OR use:
App Service → Deployment Center → Local Git
Step 10.3 - Set Startup Commands
text

Each App Service → Configuration → General Settings
Startup Command:
Service	Startup Command
Frontend	npx serve -s build -l 3000
Gateway	node server.js
User Service	node server.js
Restaurant Service	node server.js
Order Service	node server.js
PHASE 11: Configure CORS
Step 11.1 - Gateway CORS Settings
text

app-foodapp-gateway → CORS (left menu)
Allowed Origins → Add:
https://app-foodapp-frontend.azurewebsites.net
Step 11.2 - Each Backend Service
text

Each of user/restaurant/order services → CORS
Add: https://app-foodapp-gateway.azurewebsites.net
PHASE 12: Verify Private Endpoint Connection
Step 12.1 - Check Private Endpoint Status
text

Portal → "cosmos-foodapp-prod" Cosmos DB account
Left Menu → "Private endpoint connections"

Status should show: ✅ Approved
Step 12.2 - Verify DNS Resolution
text

App Service → "Advanced Tools (Kudu)" → Debug Console → CMD
Type: nslookup cosmos-foodapp-prod.mongo.cosmos.azure.com

Expected result: Should resolve to 10.0.2.x (private IP)
NOT a public IP address
PHASE 13: Network Security Configuration
Step 13.1 - Create Network Security Group for DB Subnet
text

Portal → Search "Network Security Groups" → "+ Create"
Field	Value
Resource Group	rg-foodapp-prod
Name	nsg-database
Region	East US
Step 13.2 - Add Inbound Rules
text

nsg-database → Inbound Security Rules → "+ Add"
Priority	Name	Source	Destination	Port	Action
100	allow-appservice	10.0.1.0/24	10.0.2.0/24	10255, 10256	Allow
4096	deny-all	*	*	*	Deny
Step 13.3 - Associate NSG to Database Subnet
text

nsg-database → Subnets → "+ Associate"
Virtual Network: vnet-foodapp
Subnet: snet-database
Click OK
Final Architecture Verification Checklist
text

✅ RESOURCE GROUP
   └── rg-foodapp-prod (East US)

✅ VIRTUAL NETWORK  
   └── vnet-foodapp (10.0.0.0/16)
       ├── snet-appservice (10.0.1.0/24) [Delegated to Web]
       └── snet-database (10.0.2.0/24) [Private Endpoints]

✅ COSMOS DB
   └── cosmos-foodapp-prod
       ├── Private Endpoint → snet-database
       ├── users-db → users collection
       ├── restaurants-db → restaurants collection
       └── orders-db → orders collection

✅ PRIVATE DNS ZONE
   └── privatelink.mongo.cosmos.azure.com
       └── Linked to vnet-foodapp

✅ APP SERVICE PLAN
   └── asp-foodapp-prod (P1v2, Linux)

✅ APP SERVICES (all with VNet Integration → snet-appservice)
   ├── app-foodapp-frontend (React)
   ├── app-foodapp-gateway (Port 5000)
   ├── app-foodapp-user (Port 5001)
   ├── app-foodapp-restaurant (Port 5002)
   └── app-foodapp-order (Port 5003)

✅ NETWORK SECURITY GROUP
   └── nsg-database → associated to snet-database
Troubleshooting Common Issues
text

❌ Problem: Cannot connect to Cosmos DB
✅ Fix: Check VNet Integration "Route All" is ON
        Verify Private DNS Zone linked to VNet
        Check NSG rules allow port 10255

❌ Problem: Services can't talk to each other  
✅ Fix: Use full azurewebsites.net URLs
        Both services on same App Service Plan

❌ Problem: Frontend CORS errors
✅ Fix: Add frontend URL to gateway CORS settings
        Check REACT_APP_API_URL env variable

❌ Problem: VNet Integration option grayed out
✅ Fix: App Service Plan must be P1v2 or higher
        Upgrade your plan tier first




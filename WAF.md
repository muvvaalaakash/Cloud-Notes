# 🔐 Web Application Security Guide
### For Azure DevOps Engineers

---

# 📋 Table of Contents

```
1. WAF vs NSG vs ASG vs Firewall
2. Layer 4 vs Layer 7
3. When to Implement WAF
4. OWASP Top 10
5. SQL Injection
6. Cross-Site Scripting (XSS)
7. Azure DevOps Engineer Responsibilities
```

---

# ═══════════════════════════════════
# SECTION 1: WAF vs NSG vs ASG vs FIREWALL
# ═══════════════════════════════════

---

## 🔎 Overview

As an **Azure DevOps Engineer**, you will work with multiple security layers to protect your infrastructure and applications. Understanding the difference between these tools is critical for designing secure pipelines and architectures.

---

## 🛡️ WAF (Web Application Firewall)

### What It Is:
WAF is a security solution that monitors, filters, and blocks HTTP/HTTPS traffic to and from web applications.

### What It Protects:
- Public-facing web applications
- REST APIs
- Microservices exposed to internet

### How It Works:
WAF inspects every HTTP request and response at **Layer 7 (Application Layer)** and blocks malicious content before it reaches your application.

### Azure DevOps Context:
```
When your team deploys a web application through 
Azure DevOps Pipeline:

CI/CD Pipeline deploys app
       ↓
App sits behind Application Gateway
       ↓
WAF policy attached to Application Gateway
       ↓
WAF protects app from OWASP attacks
```

### Where You Use WAF in Azure:
- Azure Application Gateway (WAF Tier)
- Azure Front Door (WAF Policy)
- Azure CDN with WAF

### DevOps Engineer Responsibility:
✅ Configure WAF policies during infrastructure setup  
✅ Define WAF rules in Terraform/Bicep (IaC)  
✅ Enable WAF in Detection or Prevention mode  
✅ Monitor WAF logs in Azure Monitor  
✅ Review WAF alerts in Azure Sentinel  

---

## 🛡️ NSG (Network Security Group)

### What It Is:
NSG is a basic network-level firewall that controls inbound and outbound traffic to Azure resources based on rules.

### What It Protects:
- Azure Virtual Machines
- Subnets
- Network Interfaces (NICs)

### How It Works:
NSG works at **Layer 3 and Layer 4**. It filters traffic based on:
- Source IP address
- Destination IP address
- Port number
- Protocol (TCP/UDP)

### Azure DevOps Context:
```
DevOps Pipeline deploys VM
       ↓
NSG attached to VM's NIC or Subnet
       ↓
NSG rules control who can reach VM
       ↓
Example: Only allow port 22 from bastion host
```

### DevOps Engineer Responsibility:
✅ Define NSG rules in IaC (Terraform/Bicep)  
✅ Apply NSG to subnets and NICs  
✅ Review NSG flow logs  
✅ Restrict unnecessary ports  
✅ Follow least privilege principle  

---

## 🛡️ ASG (Application Security Group)

### What It Is:
ASG is a logical grouping of Azure virtual machines used to simplify NSG rule management.

### What It Does:
Instead of managing individual IP addresses in NSG rules, you group VMs by their role and reference the group in NSG rules.

### Azure DevOps Context:
```
DevOps Team manages:
Web Servers → ASG-Web
App Servers → ASG-App
DB Servers  → ASG-DB

NSG Rule:
Allow ASG-Web to talk to ASG-App on port 8080
Allow ASG-App to talk to ASG-DB on port 1433

When new VM is added to ASG-Web:
Rules automatically apply → No manual IP management
```

### DevOps Engineer Responsibility:
✅ Create ASGs for each application tier  
✅ Define ASG-based NSG rules in IaC  
✅ Maintain ASG membership as VMs scale  
✅ Simplify security rule management  

---

## 🛡️ Azure Firewall

### What It Is:
Azure Firewall is a fully managed, cloud-native network security service that protects your entire Azure Virtual Network.

### What It Protects:
- Entire VNet
- Cross-VNet traffic
- Outbound internet traffic
- Inbound traffic

### How It Works:
Azure Firewall works at **Layer 3, 4, and partially Layer 7**. It provides:
- Network rules (IP/Port based)
- Application rules (FQDN based)
- DNAT rules (inbound traffic)
- Threat intelligence filtering

### Azure DevOps Context:
```
All traffic in Azure VNet flows through Azure Firewall

DevOps Pipeline → Deploys resources in VNet
                         ↓
All outbound traffic → Azure Firewall
                         ↓
Firewall checks rules
                         ↓
Allows only approved destinations
```

### DevOps Engineer Responsibility:
✅ Deploy Azure Firewall using IaC  
✅ Configure UDR (User Defined Routes)  
✅ Define application and network rules  
✅ Monitor firewall logs  
✅ Integrate with Azure Monitor and Sentinel  

---

## 📊 Comparison Table

| Feature | WAF | NSG | ASG | Azure Firewall |
|---------|-----|-----|-----|----------------|
| OSI Layer | Layer 7 | Layer 3/4 | N/A | Layer 3/4/7 |
| Scope | Web Apps | VM/Subnet | VM Grouping | Entire VNet |
| Inspects HTTP | ✅ Yes | ❌ No | ❌ No | Limited |
| IP/Port Filter | ❌ No | ✅ Yes | ❌ No | ✅ Yes |
| Simplifies Rules | ❌ No | ❌ No | ✅ Yes | ❌ No |
| Central Control | ❌ No | ❌ No | ❌ No | ✅ Yes |
| Azure Service | App Gateway | All Resources | VM Groups | Entire VNet |
| DevOps Use | App Protection | VM Protection | Rule Management | Network Control |

---

## 🏗️ Complete Azure Architecture

```
Internet Traffic
       ↓
Azure Front Door (WAF)      ← Global Layer 7 protection
       ↓
Application Gateway (WAF)   ← Regional Layer 7 protection
       ↓
Azure Firewall              ← Network Level protection
       ↓
NSG (Subnet Level)          ← Subnet Level protection
       ↓
NSG (NIC Level)             ← VM Level protection
       ↓
Virtual Machine (ASG)       ← Logical VM grouping
       ↓
Your Application
```

---

# ═══════════════════════════════════
# SECTION 2: LAYER 4 vs LAYER 7
# ═══════════════════════════════════

---

## 🔎 OSI Model Quick Reference

```
Layer 7 → Application   (HTTP, HTTPS, DNS, FTP)
Layer 6 → Presentation  (SSL/TLS)
Layer 5 → Session       (Session Management)
Layer 4 → Transport     (TCP, UDP, Ports)
Layer 3 → Network       (IP Addresses)
Layer 2 → Data Link     (MAC Addresses)
Layer 1 → Physical      (Network Hardware)
```

---

## 🔵 Layer 4 - Transport Layer

### What It Understands:
- Source IP address
- Destination IP address
- Source port number
- Destination port number
- Protocol (TCP or UDP)

### What It Does NOT Understand:
- Content of the request
- URL being accessed
- HTTP method used
- What data is being sent

### Security at Layer 4:
Layer 4 security tools only allow or block based on IP addresses and port numbers. They cannot inspect the actual content of the traffic.

### Azure Services at Layer 4:
- Azure Load Balancer
- Network Security Groups (NSG)
- Azure Firewall (Network Rules)

### DevOps Engineer Context:
```
Pipeline deploys application behind Azure Load Balancer

Load Balancer operates at Layer 4:
- Receives traffic on Port 443
- Routes to backend VM pool
- Does NOT inspect HTTP content
- Just balances based on IP and Port

DevOps Engineer manages:
✅ Load Balancer configuration in IaC
✅ Health probe settings
✅ Backend pool management
✅ NSG rules for allowed ports
```

---

## 🟢 Layer 7 - Application Layer

### What It Understands:
- Complete HTTP/HTTPS request
- URL and path
- HTTP method (GET, POST, PUT, DELETE)
- Request headers
- Cookies
- Request body and payload
- Response content

### Why Layer 7 is Important for DevOps:
Layer 7 allows intelligent routing and security decisions based on actual application content.

### Azure Services at Layer 7:
- Azure Application Gateway
- Azure Front Door
- WAF (Web Application Firewall)
- Azure API Management

### DevOps Engineer Context:
```
Pipeline deploys microservices application

Application Gateway operates at Layer 7:
- Receives HTTPS request
- Reads the URL path
- Routes /api/* → API backend pool
- Routes /web/* → Web backend pool
- Routes /admin/* → Admin backend pool
- WAF inspects each request for attacks

DevOps Engineer manages:
✅ Application Gateway configuration
✅ Path-based routing rules
✅ WAF policy in IaC
✅ SSL certificate management
✅ Backend health monitoring
```

---

## 📊 Layer 4 vs Layer 7 Comparison

| Feature | Layer 4 | Layer 7 |
|---------|---------|---------|
| Works With | IP + Port | HTTP Content |
| Speed | Faster | Slightly Slower |
| Intelligence | Low | High |
| SSL Termination | ❌ No | ✅ Yes |
| Path-based Routing | ❌ No | ✅ Yes |
| Content Inspection | ❌ No | ✅ Yes |
| Azure Service | Load Balancer | App Gateway |
| Security Level | Basic | Advanced |
| DevOps Usage | Traffic Distribution | Smart Routing + Security |

---

## 🔐 Security Concerns at Each Layer

### Layer 4 Security Concerns:

| Attack | Description | Azure Protection |
|--------|-------------|------------------|
| SYN Flood | Overwhelms server with connection requests | Azure DDoS Protection |
| UDP Flood | Floods with UDP packets | Azure DDoS Protection |
| Port Scanning | Scans for open ports | NSG Rules |
| IP Spoofing | Fakes source IP address | Azure Firewall |

### Layer 7 Security Concerns:

| Attack | Description | Azure Protection |
|--------|-------------|------------------|
| SQL Injection | Malicious SQL in input fields | WAF Rules |
| XSS | Malicious scripts injected | WAF Rules |
| DDoS HTTP Flood | Millions of HTTP requests | WAF + Front Door |
| Bot Attacks | Automated malicious traffic | WAF Bot Protection |
| CSRF | Forged user requests | WAF + App Controls |

---

## 🚦 Traffic Routing

### Layer 4 Routing Flow:
```
User Request
     ↓
Azure Load Balancer
     ↓
Checks: Destination IP + Port
     ↓
Routes to: VM1 or VM2 or VM3
(Round Robin / Least Connection)
     ↓
No content inspection
```

### Layer 7 Routing Flow:
```
User Request → https://example.com/api/products
     ↓
Azure Application Gateway
     ↓
Reads: URL Path = /api/products
     ↓
WAF: Checks for malicious content
     ↓
Routing Rule: /api/* → API Backend Pool
     ↓
Routes to: API Server Pool
     ↓
Full content inspection happened
```

---

# ═══════════════════════════════════
# SECTION 3: WHEN TO IMPLEMENT WAF
# ═══════════════════════════════════

---

## 🔎 Overview

As an Azure DevOps Engineer, you are responsible for deciding when and how to implement WAF as part of your infrastructure deployment.

---

## ✅ Scenarios When You MUST Implement WAF

### Scenario 1: Public Web Application Deployment
```
Your pipeline deploys a web application
accessible from the internet

You MUST implement WAF when:
→ Application has user login
→ Application accepts user input
→ Application connects to database
→ Application handles any user data
```

### Scenario 2: API Gateway Exposure
```
Your pipeline deploys REST APIs
accessible from external clients

You MUST implement WAF when:
→ APIs are publicly accessible
→ APIs accept request body/parameters
→ APIs return sensitive data
→ APIs connect to backend databases
```

### Scenario 3: Compliance Requirements
```
Your organization needs compliance with:
→ PCI-DSS (Payment Card Industry)
→ HIPAA (Healthcare)
→ ISO 27001
→ SOC 2

WAF is mandatory for these certifications
```

### Scenario 4: E-Commerce or Financial Application
```
Application handles:
→ Credit card information
→ Bank account details
→ Personal financial data
→ User payment flows

WAF is critical here
```

### Scenario 5: Multi-Tenant SaaS Application
```
Your application serves multiple customers
Attack on one customer affects all

WAF protects entire application layer
```

---

## 🚫 When WAF is NOT Needed

```
❌ Internal-only application
   (No internet exposure)

❌ Non-HTTP traffic
   (Database servers, file servers)

❌ Development/Test environments
   (Not exposed to internet)
   Note: Still good practice to test WAF here

❌ Pure network traffic
   (NSG and Firewall are sufficient)
```

---

## 🔧 WAF in Azure DevOps Pipeline

### DevOps Engineer Responsibilities:

✅ **Infrastructure as Code**
- Define WAF policies in Terraform or Bicep
- Version control WAF rules in Git repository
- Deploy WAF through CI/CD pipeline

✅ **WAF Modes**
```
Detection Mode:
→ WAF logs threats but does NOT block
→ Used in development and testing
→ Used when first implementing WAF
→ Review logs to understand traffic

Prevention Mode:
→ WAF actively blocks threats
→ Used in production
→ Blocks malicious requests immediately
```

✅ **WAF Policy Management**
- Create custom WAF rules for application-specific needs
- Manage OWASP rule sets
- Handle false positives by tuning rules
- Review and update rules regularly

✅ **Monitoring and Alerting**
- Configure WAF diagnostic logs
- Send logs to Log Analytics Workspace
- Create alerts in Azure Monitor
- Review in Azure Sentinel

---

## 🏗️ WAF Implementation Architecture

```
DevOps Pipeline
     ↓
Deploys Infrastructure (Terraform/Bicep)
     ↓
Creates:
→ Application Gateway (WAF Tier)
→ WAF Policy (OWASP Rules)
→ Backend Pool (VMs/App Services)
→ Listener (HTTP/HTTPS)
→ Routing Rules
     ↓
Application deployed to Backend Pool
     ↓
WAF Policy attached to Application Gateway
     ↓
All traffic inspected by WAF
```

---

# ═══════════════════════════════════
# SECTION 4: OWASP TOP 10
# ═══════════════════════════════════

---

## 🔎 What is OWASP?

**OWASP** stands for **Open Web Application Security Project**.

It is a non-profit organization dedicated to improving software security. They publish the **OWASP Top 10** — a list of the most critical web application security risks — updated every few years.

As an **Azure DevOps Engineer**, understanding OWASP is important because:
- WAF policies are based on OWASP rules
- You need to configure OWASP rule sets in Azure
- Security reviews reference OWASP standards
- Compliance requirements mention OWASP Top 10

---

## 🔥 OWASP Top 10 (2021 Edition)

---

### 🔴 A01 - Broken Access Control

**What It Is:**
Users can access data or perform actions they are not authorized for.

**Real Examples:**
- Normal user accessing admin dashboard
- User viewing another user's account by changing ID in URL
- Accessing files outside intended directory

**DevOps Engineer Impact:**
```
During deployment:
→ Ensure RBAC is configured correctly
→ Review role assignments in pipeline
→ Validate that dev has no production access
→ Implement just-in-time access for sensitive resources
→ WAF rules to block unauthorized path access
```

**Azure Controls:**
- Azure RBAC
- Azure AD Conditional Access
- WAF Custom Rules
- Azure Policy

---

### 🔴 A02 - Cryptographic Failures

**What It Is:**
Sensitive data is not properly encrypted or protected.

**Real Examples:**
- Passwords stored without proper hashing
- Using HTTP instead of HTTPS
- Storing secrets in plain text
- Using outdated encryption algorithms

**DevOps Engineer Impact:**
```
In CI/CD Pipeline:
→ Never store secrets in pipeline variables as plain text
→ Use Azure Key Vault for all secrets
→ Enforce HTTPS on all applications
→ Enable encryption at rest for storage accounts
→ Use managed identities instead of passwords
→ Rotate secrets regularly through pipeline automation
```

**Azure Controls:**
- Azure Key Vault
- Azure Key Vault references in pipelines
- Managed Identities
- Storage encryption
- SSL/TLS enforcement

---

### 🔴 A03 - Injection

**What It Is:**
Attacker injects malicious code (SQL, commands, scripts) that gets executed by the application.

**Types:**
- SQL Injection
- Command Injection
- NoSQL Injection
- LDAP Injection

**DevOps Engineer Impact:**
```
In Application Deployment:
→ WAF rules detect and block injection patterns
→ Enable WAF OWASP rule set for injection protection
→ Security testing in pipeline (SAST/DAST tools)
→ Code review gates before deployment
→ Database accounts with least privilege
→ Input validation enforced at application level
```

**Azure Controls:**
- WAF with OWASP Core Rule Set
- Azure SQL Threat Detection
- Azure Defender for SQL
- SAST tools in pipeline (SonarQube, Checkmarx)

---

### 🔴 A04 - Insecure Design

**What It Is:**
Application is designed without security considerations from the beginning.

**Real Examples:**
- No rate limiting on login page
- No account lockout after failed attempts
- Password reset using easily guessable questions
- No input length validation

**DevOps Engineer Impact:**
```
Security must be part of design phase:
→ Include security requirements in sprint planning
→ Threat modeling sessions before development
→ Security checklist in pull request templates
→ Architecture review for new features
→ DevSecOps approach in pipeline
→ Security gates in deployment pipeline
```

**Azure Controls:**
- Azure API Management (Rate Limiting)
- Azure AD (Account Lockout Policies)
- Azure DevOps Branch Policies
- Pipeline security gates

---

### 🔴 A05 - Security Misconfiguration

**What It Is:**
Security settings are incorrectly configured or left as default.

**Real Examples:**
- Default admin credentials not changed
- Debug mode left on in production
- Unnecessary services running
- Open cloud storage (public blob containers)
- Detailed error messages exposed

**DevOps Engineer Impact:**
```
This is DIRECTLY a DevOps Engineer responsibility:
→ Infrastructure as Code ensures consistent configuration
→ Azure Policy enforces security standards
→ No public blob containers allowed
→ Debug settings disabled in production config
→ Microsoft Defender for Cloud recommendations
→ Regular security posture reviews
→ Automated compliance checks in pipeline
→ Environment-specific configuration management
```

**Azure Controls:**
- Azure Policy
- Microsoft Defender for Cloud
- Azure Blueprints
- IaC security scanning (Checkov, tfsec)
- Azure Security Benchmark

---

### 🔴 A06 - Vulnerable and Outdated Components

**What It Is:**
Using libraries, frameworks, or software with known security vulnerabilities.

**Real Examples:**
- Log4Shell vulnerability in Log4j library
- Outdated WordPress plugins
- Old npm packages with known CVEs
- Unpatched operating systems

**DevOps Engineer Impact:**
```
This is a CORE DevOps responsibility:
→ Dependency scanning in CI/CD pipeline
→ Container image scanning before deployment
→ Automated patch management
→ Base image updates in Docker builds
→ Vulnerability scanning with Microsoft Defender
→ Software composition analysis (SCA) in pipeline
→ Alert on new CVEs for used components
→ Regular dependency updates
```

**Azure Controls:**
- Microsoft Defender for Containers
- Microsoft Defender for Servers
- Azure Container Registry scanning
- Pipeline dependency scanning tools
- Azure Update Management

---

### 🔴 A07 - Identification and Authentication Failures

**What It Is:**
Weak authentication mechanisms that can be exploited.

**Real Examples:**
- No Multi-Factor Authentication
- Weak password policies
- Session tokens not expiring
- Credentials stored in code

**DevOps Engineer Impact:**
```
→ Enforce MFA for all Azure DevOps users
→ Use managed identities for pipeline authentication
→ No hardcoded credentials in pipeline code
→ Use service principals with minimum permissions
→ Rotate service principal secrets regularly
→ Azure AD Conditional Access policies
→ Pipeline accesses Key Vault for credentials
→ Regular access review of pipeline service connections
```

**Azure Controls:**
- Azure Active Directory MFA
- Conditional Access Policies
- Managed Identities
- Azure Key Vault
- Privileged Identity Management (PIM)

---

### 🔴 A08 - Software and Data Integrity Failures

**What It Is:**
Trusting unverified software updates, plugins, or data in CI/CD pipelines.

**Real Examples:**
- SolarWinds supply chain attack
- Malicious packages in npm/PyPI
- Compromised build pipeline
- Unsigned software packages

**DevOps Engineer Impact:**
```
This directly affects CI/CD pipelines:
→ Verify integrity of pipeline tasks and extensions
→ Use approved marketplace tasks only
→ Pin specific versions of pipeline tasks
→ Code signing for releases
→ Secure build environments
→ Restrict pipeline permissions
→ Review third-party dependencies
→ Separate build and deployment environments
→ Audit pipeline changes through Git history
```

**Azure Controls:**
- Azure DevOps Auditing
- Branch protection policies
- Pipeline approval gates
- Azure Artifacts with vulnerability scanning
- Signed pipeline artifacts

---

### 🔴 A09 - Security Logging and Monitoring Failures

**What It Is:**
Not capturing or monitoring security-relevant events, leaving attacks undetected.

**Real Examples:**
- No logs for failed login attempts
- Breach undetected for months
- No alerts for suspicious activity
- Logs not protected from tampering

**DevOps Engineer Impact:**
```
This is a KEY DevOps responsibility:
→ Enable diagnostic logs for all Azure resources
→ Send logs to central Log Analytics Workspace
→ Configure Azure Monitor alerts
→ Set up Azure Sentinel for SIEM
→ Pipeline deployment logs retained
→ Alert on failed deployments
→ Monitor WAF blocked requests
→ Track unusual resource access patterns
→ Review Azure Activity logs regularly
→ Implement log retention policies
```

**Azure Controls:**
- Azure Monitor
- Log Analytics Workspace
- Azure Sentinel
- Application Insights
- Azure Activity Log
- Diagnostic Settings on all resources

---

### 🔴 A10 - Server-Side Request Forgery (SSRF)

**What It Is:**
Attacker forces the server to make requests to unintended internal or external locations.

**Real Examples:**
- Accessing cloud metadata endpoint
- Reaching internal databases through server
- Accessing internal admin interfaces
- Bypassing network security controls

**DevOps Engineer Impact:**
```
→ WAF rules to block SSRF patterns
→ Network segmentation through NSG
→ Block access to metadata endpoints
→ Outbound traffic control through Azure Firewall
→ No direct internet access for app servers
→ Use private endpoints for Azure services
→ Implement egress filtering
```

**Azure Controls:**
- Azure WAF SSRF rules
- Private Endpoints
- Azure Firewall outbound rules
- NSG outbound rules
- Service Endpoints

---

## 📊 OWASP Top 10 Summary for DevOps Engineers

| # | Vulnerability | DevOps Action | Azure Tool |
|---|--------------|---------------|------------|
| A01 | Broken Access Control | Configure RBAC properly | Azure RBAC, WAF |
| A02 | Cryptographic Failures | Use Key Vault, enforce HTTPS | Key Vault, App Gateway |
| A03 | Injection | Enable WAF OWASP rules | WAF, Defender for SQL |
| A04 | Insecure Design | Security in sprint planning | Azure Policy, API Management |
| A05 | Misconfiguration | IaC + Azure Policy | Defender for Cloud, Policy |
| A06 | Outdated Components | Dependency scanning in pipeline | Defender for Containers |
| A07 | Auth Failures | MFA + Managed Identities | Azure AD, PIM |
| A08 | Integrity Failures | Secure pipelines + signing | DevOps Auditing, Artifacts |
| A09 | Logging Failures | Centralized logging + alerts | Sentinel, Monitor |
| A10 | SSRF | WAF rules + Private Endpoints | WAF, Private Endpoints |

---

# ═══════════════════════════════════
# SECTION 5: SQL INJECTION (Deep Dive)
# ═══════════════════════════════════

---

## 🔎 What is SQL Injection?

SQL Injection is a web security attack where an attacker inserts malicious SQL commands into input fields. These commands are then executed by the application's database, potentially exposing or destroying data.

---

## 🎯 How SQL Injection Works

### Normal User Flow:
```
User enters:
Username: admin
Password: password123

Application queries database:
→ Find user where username is admin
→ AND password is password123
→ Returns user record
→ Login successful
```

### Attacker's Flow:
```
Attacker enters:
Username: admin' OR 1=1 --
Password: anything

Application queries database:
→ Find user where username is admin
→ OR where 1 equals 1 (always true)
→ Rest of query commented out
→ Returns ALL users
→ Login bypassed without password
```

---

## 💥 Types of SQL Injection

### 1. Authentication Bypass
Attacker bypasses login without knowing password.

### 2. Data Extraction
Attacker reads entire database contents including:
- Usernames and passwords
- Credit card numbers
- Personal information
- Business data

### 3. Data Modification
Attacker changes database records:
- Modifies prices in e-commerce
- Changes account balances
- Updates user permissions

### 4. Data Deletion
Attacker deletes entire tables or databases.

### 5. Blind SQL Injection
Attacker cannot see direct output but infers data through:
- True/False responses
- Time delays in responses

---

## 🔧 DevOps Engineer Responsibilities

### In CI/CD Pipeline:
```
Stage 1: Code Commit
→ SAST (Static Analysis) scans for SQL injection patterns
→ SonarQube or Checkmarx identifies vulnerable code
→ Build fails if critical SQL injection vulnerability found

Stage 2: Build Stage
→ Dependency scan for vulnerable database libraries
→ Container image scan if using containers

Stage 3: Test Stage
→ DAST (Dynamic Analysis) tests running application
→ OWASP ZAP scans for SQL injection
→ Penetration test results reviewed

Stage 4: Pre-Production
→ WAF enabled in detection mode
→ Review WAF logs for SQL injection attempts
→ Tune WAF rules if false positives

Stage 5: Production
→ WAF in prevention mode
→ Azure Defender for SQL enabled
→ Database audit logging enabled
→ Alerts configured for suspicious queries
```

### Infrastructure Controls:
```
✅ Deploy WAF with OWASP Core Rule Set
✅ Enable Azure Defender for SQL
✅ Implement database least privilege access
✅ Enable SQL Threat Detection
✅ Configure alerts for anomalous database activity
✅ Regular vulnerability assessments
✅ Database activity monitoring
```

---

## 🛡️ Azure Protection for SQL Injection

| Azure Service | How It Protects |
|---------------|-----------------|
| WAF (App Gateway) | Blocks SQL injection patterns in HTTP requests |
| Azure Defender for SQL | Detects unusual database queries |
| SQL Threat Detection | Alerts on SQL injection attempts |
| Azure Monitor | Logs all database access |
| Microsoft Sentinel | Correlates SQL injection events |

---

## 📊 SQL Injection Impact Assessment

| Impact Area | Description | Severity |
|-------------|-------------|----------|
| Authentication | Complete bypass of login | Critical |
| Data Breach | All data exposed | Critical |
| Data Loss | Database deleted | Critical |
| Compliance | GDPR/PCI violation | Critical |
| Reputation | Customer trust lost | High |
| Financial | Fines and recovery costs | High |

---

# ═══════════════════════════════════
# SECTION 6: CROSS-SITE SCRIPTING (XSS)
# ═══════════════════════════════════

---

## 🔎 What is Cross-Site Scripting (XSS)?

XSS is a web security vulnerability where an attacker injects malicious JavaScript into a trusted website. This script then runs in the browsers of other users visiting the site, allowing attackers to steal data, hijack sessions, or perform actions on behalf of users.

Key Difference from SQL Injection:
- SQL Injection targets the **SERVER and DATABASE**
- XSS targets the **USER'S BROWSER**

---

## 🎯 How XSS Works

### Normal Comment Section:
```
User posts comment:
"Great article! Very helpful."

Other users see:
"Great article! Very helpful."
```

### Attacker Injects XSS:
```
Attacker posts:
<script>steal user cookie and send to attacker</script>

Application stores this without sanitization

Other users open page:
→ Script runs automatically in their browser
→ Their session cookie sent to attacker
→ Attacker logs in as victim
→ Account hijacked
```

---

## 🔥 Types of XSS

### 1. Stored XSS (Persistent) - Most Dangerous
```
Where it happens:
→ Comment sections
→ User profiles
→ Forum posts
→ Product reviews

How it works:
Attacker posts malicious script
→ Script saved in database
→ Every user who views the page
→ Script executes in their browser
→ Affects ALL users, not just one
```

### 2. Reflected XSS
```
Where it happens:
→ Search results pages
→ Error messages
→ URL parameters

How it works:
Attacker crafts malicious URL
→ Shares link with victim
→ Victim clicks link
→ Server reflects script in response
→ Script executes in victim's browser
→ Affects only users who click the link
```

### 3. DOM-Based XSS
```
Where it happens:
→ Single Page Applications (React, Angular, Vue)
→ Client-side JavaScript

How it works:
Attacker manipulates page URL or input
→ JavaScript on page processes malicious input
→ Script executes in browser
→ No server involvement needed
```

---

## 💥 What Can Attacker Do with XSS?

```
🔴 Session Hijacking
→ Steal session cookies
→ Log in as victim
→ Access account without password

🔴 Account Takeover
→ Change victim's password
→ Change victim's email
→ Take over account permanently

🔴 Data Theft
→ Steal personal information shown on page
→ Capture credit card details
→ Read private messages

🔴 Malware Distribution
→ Redirect users to malicious sites
→ Download malware on victim computers
→ Phishing attacks

🔴 Website Defacement
→ Change website content
→ Display fake messages
→ Damage brand reputation

🔴 Keylogging
→ Record everything user types
→ Capture passwords and sensitive data
```

---

## 🔧 DevOps Engineer Responsibilities

### In CI/CD Pipeline:
```
Stage 1: Code Review
→ Review output encoding in templates
→ Check JavaScript code for DOM manipulation
→ Verify Content Security Policy headers

Stage 2: SAST Scanning
→ Static analysis detects potential XSS
→ Check for unescaped output
→ Identify unsafe JavaScript practices

Stage 3: DAST Testing
→ OWASP ZAP tests for XSS vulnerabilities
→ Automated XSS payload testing
→ Review test results before deployment

Stage 4: Infrastructure
→ WAF with XSS rule set enabled
→ Security headers configured
→ CSP policy defined and enforced

Stage 5: Production Monitoring
→ WAF logs reviewed for XSS attempts
→ Application Insights for anomalies
→ Alert on suspicious JavaScript activity
```

### Infrastructure Controls:
```
✅ Enable WAF XSS protection rules
✅ Configure HTTP security headers
✅ Enable Content Security Policy
✅ HttpOnly cookie flag enabled
✅ Secure cookie flag enabled
✅ Monitor WAF blocked requests
✅ Application Insights monitoring
```

---

## 🛡️ Azure Protection for XSS

| Azure Service | How It Protects |
|---------------|-----------------|
| WAF (App Gateway) | Blocks XSS patterns in requests |
| Azure Front Door WAF | Global XSS protection |
| Application Gateway | Security headers enforcement |
| Azure CDN with WAF | Edge-level XSS protection |
| Application Insights | Detects unusual client-side behavior |

---

## 📊 SQL Injection vs XSS Comparison

| Feature | SQL Injection | XSS |
|---------|---------------|-----|
| Target | Database/Server | User's Browser |
| Language | SQL Commands | JavaScript |
| Goal | Steal/Modify DB Data | Steal Session/Data |
| Affects | Server | Other Users |
| Layer | Backend | Frontend |
| Stored | In Database | In Page/URL |
| Detected by | WAF/DB Monitoring | WAF/Browser Security |
| Prevention | Parameterized Queries | Output Encoding |
| Azure Tool | WAF + Defender for SQL | WAF + CSP Headers |
| Severity | Critical | High to Critical |

---

# ═══════════════════════════════════
# SECTION 7: AZURE DEVOPS ENGINEER
# OVERALL SECURITY RESPONSIBILITIES
# ═══════════════════════════════════

---

## 🔎 DevSecOps Approach

As an Azure DevOps Engineer, security is not just an afterthought — it must be integrated into every stage of the pipeline.

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ↓      ↓       ↓      ↓       ↓         ↓        ↓         ↓
Security Security SAST  DAST   Approval  WAF    Defender  Sentinel
Review  Scan    Scan   Test   Gates     NSG    Cloud     Alerts
```

---

## 🔧 Security at Each Pipeline Stage

### 1. Plan Stage
```
✅ Threat modeling for new features
✅ Security requirements defined
✅ OWASP Top 10 review
✅ Compliance requirements identified
✅ Security user stories in backlog
```

### 2. Code Stage
```
✅ Pre-commit security hooks
✅ Secret detection (no credentials in code)
✅ Peer code review for security
✅ Branch protection policies
✅ Signed commits
```

### 3. Build Stage
```
✅ SAST (Static Application Security Testing)
✅ Dependency vulnerability scanning
✅ Container image scanning
✅ IaC security scanning (Checkov/tfsec)
✅ Build fails on critical vulnerabilities
```

### 4. Test Stage
```
✅ DAST (Dynamic Application Security Testing)
✅ OWASP ZAP automated scanning
✅ Penetration testing
✅ WAF rule testing
✅ Security regression testing
```

### 5. Release Stage
```
✅ Security approval gates
✅ Compliance sign-off
✅ Vulnerability report review
✅ WAF policy review
✅ Production readiness checklist
```

### 6. Deploy Stage
```
✅ WAF enabled in Prevention mode
✅ NSG rules applied
✅ Azure Firewall rules configured
✅ Managed identities used
✅ Key Vault references configured
✅ Private endpoints configured
```

### 7. Operate Stage
```
✅ Patch management automated
✅ Vulnerability assessments regular
✅ Access reviews quarterly
✅ Secret rotation automated
✅ Compliance monitoring
```

### 8. Monitor Stage
```
✅ WAF logs to Log Analytics
✅ Azure Monitor alerts configured
✅ Microsoft Sentinel SIEM active
✅ Application Insights monitoring
✅ Azure Defender alerts reviewed
✅ Security dashboard maintained
```

---

## 📋 Security Checklist for Azure DevOps Engineers

### Infrastructure Security:
```
✅ WAF deployed and in Prevention mode
✅ NSG rules follow least privilege
✅ ASG used for VM grouping
✅ Azure Firewall for centralized control
✅ Private endpoints for PaaS services
✅ DDoS Protection enabled
✅ Azure Defender for Cloud enabled
```

### Pipeline Security:
```
✅ No secrets in pipeline variables
✅ Key Vault integration for secrets
✅ Managed identities for authentication
✅ SAST tools integrated
✅ DAST tools in test stage
✅ Container scanning enabled
✅ IaC scanning enabled
✅ Approval gates configured
```

### Application Security:
```
✅ HTTPS enforced everywhere
✅ OWASP Top 10 addressed
✅ WAF OWASP rule set enabled
✅ Security headers configured
✅ Certificate management automated
✅ SQL injection protection
✅ XSS protection enabled
```

### Monitoring Security:
```
✅ All diagnostic logs enabled
✅ Central Log Analytics Workspace
✅ Azure Sentinel configured
✅ Alert rules defined
✅ Incident response plan documented
✅ Regular security reviews scheduled
```

---

## 🏆 Interview Summary for Azure DevOps Engineers

### On WAF vs NSG vs ASG vs Firewall:
> WAF protects web applications at Layer 7 against OWASP attacks. NSG controls network traffic at Layer 3/4 based on IP and port. ASG logically groups VMs to simplify NSG rule management. Azure Firewall provides centralized network protection for the entire VNet. As a DevOps engineer, I deploy and manage all these through Infrastructure as Code and monitor them through Azure Monitor and Sentinel.

### On Layer 4 vs Layer 7:
> Layer 4 operates at transport level using IP and port — Azure Load Balancer uses this. Layer 7 operates at application level understanding HTTP content — Application Gateway and WAF use this. I configure path-based routing at Layer 7 and basic load balancing at Layer 4 through IaC deployment.

### On OWASP Top 10:
> OWASP Top 10 are the most critical web application security risks. As a DevOps engineer, I address these through WAF policies, pipeline security scanning, Azure Defender, proper RBAC, Key Vault for secrets, and centralized monitoring with Sentinel.

### On SQL Injection and XSS:
> SQL injection attacks the database through malicious SQL in input fields. XSS attacks users through malicious JavaScript in web pages. I protect against both by enabling WAF with OWASP rule sets, integrating SAST/DAST scanning in pipelines, enabling Azure Defender for SQL, and monitoring through Azure Sentinel.

---

*Document Version: 1.0*
*Audience: Azure DevOps Engineers*
*Topics: WAF, NSG, ASG, Firewall, Layer 4, Layer 7, OWASP Top 10, SQL Injection, XSS*


🧠 One-Line Summary of Everything

| Topic               | One Line Summary                                                       |
| ------------------- | ---------------------------------------------------------------------- |
| WAF                 | Protects web applications from HTTP/HTTPS attacks at Layer 7           |
| NSG                 | Controls inbound and outbound traffic using IPs, ports, and protocols  |
| ASG                 | Groups VMs logically to simplify NSG rule management                   |
| Firewall            | Centralized security service that filters and monitors network traffic |
| Layer 4             | Works with TCP/UDP, IP addresses, and ports only                       |
| Layer 7             | Understands application data like HTTP requests, URLs, and headers     |
| OWASP               | Standard list of major web application security vulnerabilities        |
| SQL Injection       | Attack where malicious SQL queries manipulate databases                |
| XSS                 | Attack where malicious JavaScript executes in a user’s browser         |
| WAF Detection Mode  | Monitors and logs attacks without blocking them                        |
| WAF Prevention Mode | Detects and actively blocks malicious requests                         |

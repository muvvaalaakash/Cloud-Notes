# Complete Step-by-Step Guide: Azure Front Door → Application Gateway + WAF → App Services (Blue-Green) → Cosmos DB

## Architecture Overview
```
Azure Front Door
    ↓
Application Gateway + WAF
    ↓
App Services (Blue-Green)
├── Blue Slot: Organic Ghee (Original)
├── Green Slot: Organic Ghee (Changed CSS color)
    ↓
Cosmos DB
```

---

## PHASE 1: RESOURCE GROUP CREATION

### Step 1: Create Resource Group
1. Log in to **Azure Portal** → https://portal.azure.com
2. Search **"Resource groups"** in the top search bar
3. Click **"+ Create"**
4. **Subscription**: Select your subscription
5. **Resource group name**: `rg-organicghee-prod`
6. **Region**: `East US` (or your preferred region)
7. Click **"Review + create"**
8. Click **"Create"**

---

## PHASE 2: VIRTUAL NETWORK SETUP

### Step 2: Create Virtual Network
1. Search **"Virtual networks"** in the search bar
2. Click **"+ Create"**
3. **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Virtual network name**: `vnet-organicghee`
   - **Region**: `East US`
4. Click **"Next: IP Addresses"**
5. **IP Addresses tab**:
   - **IPv4 address space**: `10.0.0.0/16`
   - Click **"+ Add subnet"**:
     - **Subnet name**: `subnet-appgw`
     - **Subnet address range**: `10.0.1.0/24`
     - Click **"Add"**
   - Click **"+ Add subnet"** again:
     - **Subnet name**: `subnet-appservice`
     - **Subnet address range**: `10.0.2.0/24`
     - Click **"Add"**
   - Click **"+ Add subnet"** again:
     - **Subnet name**: `subnet-cosmosdb`
     - **Subnet address range**: `10.0.3.0/24`
     - Click **"Add"**
6. Click **"Next: Security"** → Leave defaults
7. Click **"Review + create"**
8. Click **"Create"**

---

## PHASE 3: COSMOS DB SETUP

### Step 3: Create Cosmos DB Account
1. Search **"Azure Cosmos DB"** in the search bar
2. Click **"+ Create"**
3. Select **"Azure Cosmos DB for NoSQL"** → Click **"Create"**
4. **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Account Name**: `cosmos-organicghee` (must be globally unique, e.g., `cosmos-organicghee-12345`)
   - **Location**: `East US`
   - **Capacity mode**: `Serverless` (for cost savings) or `Provisioned throughput`
   - **Apply Free Tier Discount**: `Apply` (if available)
5. Click **"Next: Global Distribution"** → Leave defaults
6. Click **"Next: Networking"** → Select **"All networks"** (for simplicity, we'll secure later)
7. Click **"Next: Backup Policy"** → Leave defaults
8. Click **"Next: Encryption"** → Leave defaults
9. Click **"Review + create"**
10. Click **"Create"**
11. **Wait for deployment** (takes 5-10 minutes)

### Step 4: Create Database and Container
1. Once deployed, click **"Go to resource"**
2. In the left menu, click **"Data Explorer"**
3. Click **"New Container"**
4. **Database id**: Select **"Create new"** → Enter `OrganicGheeDB`
5. **Container id**: `Products`
6. **Partition key**: `/category`
7. Click **"OK"**

### Step 5: Add Sample Data
1. In **Data Explorer**, expand **OrganicGheeDB** → **Products** → Click **"Items"**
2. Click **"New Item"**
3. Replace the JSON with:
```json
{
    "id": "1",
    "name": "Premium Organic Ghee",
    "category": "ghee",
    "price": 24.99,
    "description": "100% Pure Organic Grass-Fed Ghee",
    "weight": "500ml",
    "inStock": true
}
```
4. Click **"Save"**
5. Click **"New Item"** again and add:
```json
{
    "id": "2",
    "name": "Traditional Organic Ghee",
    "category": "ghee",
    "price": 19.99,
    "description": "Traditional Hand-Churned Organic Ghee",
    "weight": "250ml",
    "inStock": true
}
```
6. Click **"Save"**

### Step 6: Get Cosmos DB Connection String
1. In the Cosmos DB resource, click **"Keys"** in the left menu under **Settings**
2. **Copy and save** the following (you'll need these later):
   - **URI**: `https://cosmos-organicghee-12345.documents.azure.com:443/`
   - **PRIMARY KEY**: `xxxxxxxxxxxxxxx`
   - **PRIMARY CONNECTION STRING**: `AccountEndpoint=https://...`

---

## PHASE 4: APP SERVICE SETUP (BLUE-GREEN)

### Step 7: Create App Service Plan
1. Search **"App Service plans"** in the search bar
2. Click **"+ Create"**
3. **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Name**: `asp-organicghee`
   - **Operating System**: `Linux`
   - **Region**: `East US`
   - **Pricing tier**: Click **"Explore pricing plans"**
     - Select **"Standard S1"** (required for deployment slots)
     - Click **"Select"**
4. Click **"Review + create"**
5. Click **"Create"**

### Step 8: Create App Service (Blue - Production)
1. Search **"App Services"** in the search bar
2. Click **"+ Create"** → Select **"Web App"**
3. **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Name**: `app-organicghee` (must be globally unique, e.g., `app-organicghee-12345`)
   - **Publish**: `Code`
   - **Runtime stack**: `Node 18 LTS` (or your preferred runtime)
   - **Operating System**: `Linux`
   - **Region**: `East US`
   - **Linux Plan**: Select `asp-organicghee`
4. Click **"Next: Database"** → Skip
5. Click **"Next: Deployment"** → Leave defaults
6. Click **"Next: Networking"** → Leave defaults for now
7. Click **"Next: Monitoring"**:
   - **Enable Application Insights**: `Yes`
   - **Application Insights**: Click **"Create new"**
     - **Name**: `ai-organicghee`
     - Click **"OK"**
8. Click **"Review + create"**
9. Click **"Create"**
10. **Wait for deployment**

### Step 9: Configure App Service Environment Variables
1. Go to the App Service → Click **"Go to resource"**
2. In the left menu under **Settings**, click **"Environment variables"**
3. Click **"+ Add"** under **App settings**:
   - **Name**: `COSMOS_ENDPOINT`
   - **Value**: `https://cosmos-organicghee-12345.documents.azure.com:443/` (your URI)
   - Click **"Apply"**
4. Click **"+ Add"** again:
   - **Name**: `COSMOS_KEY`
   - **Value**: Your PRIMARY KEY from Step 6
   - Click **"Apply"**
5. Click **"+ Add"** again:
   - **Name**: `COSMOS_DATABASE`
   - **Value**: `OrganicGheeDB`
   - Click **"Apply"**
6. Click **"+ Add"** again:
   - **Name**: `COSMOS_CONTAINER`
   - **Value**: `Products`
   - Click **"Apply"**
7. Click **"+ Add"** again:
   - **Name**: `SLOT_NAME`
   - **Value**: `blue`
   - Click **"Apply"**
8. Click **"Apply"** at the bottom of the page
9. Click **"Confirm"** when prompted

### Step 10: Deploy Blue Application Code
1. In the App Service, click **"Advanced Tools"** in the left menu under **Development Tools**
2. Click **"Go →"** (opens Kudu)
3. Alternatively, we'll use the **App Service Editor** or **Deployment Center**

**Method: Using Azure Cloud Shell (Recommended)**

1. Click the **Cloud Shell** icon (>_) at the top of the Azure Portal
2. Select **"Bash"**
3. If prompted, create storage account
4. Run the following commands:

```bash
# Create a directory for the blue app
mkdir -p organicghee-blue
cd organicghee-blue

# Initialize Node.js project
cat > package.json << 'EOF'
{
  "name": "organic-ghee-app",
  "version": "1.0.0",
  "description": "Organic Ghee Store",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "@azure/cosmos": "^4.0.0",
    "express": "^4.18.2"
  }
}
EOF

# Create the main application file
cat > app.js << 'APPEOF'
const express = require('express');
const { CosmosClient } = require('@azure/cosmos');
const path = require('path');

const app = express();
const port = process.env.PORT || 8080;

// Cosmos DB configuration
const endpoint = process.env.COSMOS_ENDPOINT;
const key = process.env.COSMOS_KEY;
const databaseId = process.env.COSMOS_DATABASE || 'OrganicGheeDB';
const containerId = process.env.COSMOS_CONTAINER || 'Products';
const slotName = process.env.SLOT_NAME || 'blue';

let container;

async function initCosmosDB() {
    try {
        if (endpoint && key) {
            const client = new CosmosClient({ endpoint, key });
            const database = client.database(databaseId);
            container = database.container(containerId);
            console.log('Connected to Cosmos DB');
        } else {
            console.log('Cosmos DB credentials not configured, using sample data');
        }
    } catch (error) {
        console.log('Cosmos DB connection failed, using sample data:', error.message);
    }
}

async function getProducts() {
    if (container) {
        try {
            const { resources } = await container.items.readAll().fetchAll();
            return resources;
        } catch (error) {
            console.log('Error fetching from Cosmos DB:', error.message);
        }
    }
    // Fallback sample data
    return [
        { id: '1', name: 'Premium Organic Ghee', category: 'ghee', price: 24.99, description: '100% Pure Organic Grass-Fed Ghee', weight: '500ml', inStock: true },
        { id: '2', name: 'Traditional Organic Ghee', category: 'ghee', price: 19.99, description: 'Traditional Hand-Churned Organic Ghee', weight: '250ml', inStock: true }
    ];
}

app.get('/', async (req, res) => {
    const products = await getProducts();
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Organic Ghee Store</title>
        <style>
            * { margin: 0; padding: 0; box-sizing: border-box; }
            body {
                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                background-color: #faf8f5;
                color: #333;
            }
            header {
                background-color: #8B4513;
                color: white;
                padding: 20px 0;
                text-align: center;
                box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            }
            header h1 {
                font-size: 2.5em;
                margin-bottom: 5px;
            }
            .slot-badge {
                background-color: #2196F3;
                color: white;
                padding: 5px 15px;
                border-radius: 20px;
                display: inline-block;
                margin-top: 10px;
                font-size: 0.9em;
                font-weight: bold;
            }
            .container {
                max-width: 1200px;
                margin: 30px auto;
                padding: 0 20px;
            }
            .tagline {
                text-align: center;
                font-size: 1.3em;
                color: #8B4513;
                margin-bottom: 30px;
                font-style: italic;
            }
            .products-grid {
                display: grid;
                grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
                gap: 25px;
                padding: 20px 0;
            }
            .product-card {
                background: white;
                border-radius: 15px;
                padding: 30px;
                box-shadow: 0 4px 15px rgba(0,0,0,0.1);
                transition: transform 0.3s ease;
                border-top: 4px solid #DAA520;
            }
            .product-card:hover {
                transform: translateY(-5px);
            }
            .product-card h2 {
                color: #8B4513;
                margin-bottom: 10px;
                font-size: 1.4em;
            }
            .product-card .price {
                font-size: 1.8em;
                color: #DAA520;
                font-weight: bold;
                margin: 15px 0;
            }
            .product-card .description {
                color: #666;
                line-height: 1.6;
                margin-bottom: 10px;
            }
            .product-card .weight {
                color: #999;
                font-size: 0.9em;
            }
            .stock-badge {
                display: inline-block;
                padding: 5px 12px;
                border-radius: 12px;
                font-size: 0.85em;
                font-weight: bold;
                margin-top: 10px;
            }
            .in-stock {
                background-color: #e8f5e9;
                color: #2e7d32;
            }
            .buy-btn {
                display: block;
                width: 100%;
                padding: 12px;
                margin-top: 15px;
                background-color: #8B4513;
                color: white;
                border: none;
                border-radius: 8px;
                font-size: 1.1em;
                cursor: pointer;
                transition: background-color 0.3s;
            }
            .buy-btn:hover {
                background-color: #A0522D;
            }
            footer {
                text-align: center;
                padding: 20px;
                background-color: #8B4513;
                color: white;
                margin-top: 50px;
            }
        </style>
    </head>
    <body>
        <header>
            <h1>🧈 Organic Ghee Store</h1>
            <p>Pure. Natural. Traditional.</p>
            <span class="slot-badge">🔵 BLUE Deployment (Production)</span>
        </header>
        <div class="container">
            <p class="tagline">"From Farm to Table - The Purest Ghee Experience"</p>
            <div class="products-grid">
                ${products.map(p => `
                    <div class="product-card">
                        <h2>${p.name}</h2>
                        <p class="description">${p.description}</p>
                        <p class="price">$${p.price}</p>
                        <p class="weight">Weight: ${p.weight}</p>
                        <span class="stock-badge in-stock">${p.inStock ? '✅ In Stock' : '❌ Out of Stock'}</span>
                        <button class="buy-btn">Add to Cart</button>
                    </div>
                `).join('')}
            </div>
        </div>
        <footer>
            <p>&copy; 2024 Organic Ghee Store | Slot: ${slotName.toUpperCase()}</p>
        </footer>
    </body>
    </html>
    `);
});

app.get('/health', (req, res) => {
    res.json({ status: 'healthy', slot: slotName, timestamp: new Date().toISOString() });
});

app.get('/api/products', async (req, res) => {
    const products = await getProducts();
    res.json(products);
});

initCosmosDB().then(() => {
    app.listen(port, () => {
        console.log(`Organic Ghee Store (${slotName}) running on port ${port}`);
    });
});
APPEOF

# Create the zip file
zip -r organicghee-blue.zip .

# Deploy to App Service (replace with your app name)
az webapp deployment source config-zip \
  --resource-group rg-organicghee-prod \
  --name app-organicghee-12345 \
  --src organicghee-blue.zip

echo "Blue deployment completed!"
```

> ⚠️ **Replace `app-organicghee-12345`** with your actual App Service name throughout.

### Step 11: Verify Blue Deployment
1. Go back to the App Service in the portal
2. Click **"Overview"**
3. Click the **Default domain** URL (e.g., `https://app-organicghee-12345.azurewebsites.net`)
4. You should see the **Organic Ghee Store** with **brown theme (🔵 BLUE Deployment badge)**

### Step 12: Create Green Deployment Slot
1. In the App Service, click **"Deployment slots"** in the left menu under **Deployment**
2. Click **"+ Add Slot"**
3. **Name**: `green`
4. **Clone settings from**: `app-organicghee-12345` (clone from production)
5. Click **"Add"**
6. Wait for the slot to be created
7. Click on the **green** slot that appears in the list

### Step 13: Configure Green Slot Environment Variables
1. You're now in the **green slot** resource
2. Click **"Environment variables"** under **Settings**
3. Find **SLOT_NAME** and click on it:
   - Change **Value** to: `green`
   - Check the box **"Deployment slot setting"** ✅ (this makes it sticky to the slot)
   - Click **"Apply"**
4. Also make the following slot-sticky (click on each, check "Deployment slot setting"):
   - `SLOT_NAME` → ✅ Deployment slot setting
5. Click **"Apply"** at the bottom
6. Click **"Confirm"**

### Step 14: Deploy Green Application Code (Changed CSS)
1. Go back to **Cloud Shell**
2. Run the following:

```bash
# Create directory for green app
mkdir -p ~/organicghee-green
cd ~/organicghee-green

# Copy package.json
cat > package.json << 'EOF'
{
  "name": "organic-ghee-app",
  "version": "2.0.0",
  "description": "Organic Ghee Store - Green",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "@azure/cosmos": "^4.0.0",
    "express": "^4.18.2"
  }
}
EOF

# Create the GREEN version with CHANGED CSS COLORS
cat > app.js << 'APPEOF'
const express = require('express');
const { CosmosClient } = require('@azure/cosmos');
const path = require('path');

const app = express();
const port = process.env.PORT || 8080;

// Cosmos DB configuration
const endpoint = process.env.COSMOS_ENDPOINT;
const key = process.env.COSMOS_KEY;
const databaseId = process.env.COSMOS_DATABASE || 'OrganicGheeDB';
const containerId = process.env.COSMOS_CONTAINER || 'Products';
const slotName = process.env.SLOT_NAME || 'green';

let container;

async function initCosmosDB() {
    try {
        if (endpoint && key) {
            const client = new CosmosClient({ endpoint, key });
            const database = client.database(databaseId);
            container = database.container(containerId);
            console.log('Connected to Cosmos DB');
        } else {
            console.log('Cosmos DB credentials not configured, using sample data');
        }
    } catch (error) {
        console.log('Cosmos DB connection failed, using sample data:', error.message);
    }
}

async function getProducts() {
    if (container) {
        try {
            const { resources } = await container.items.readAll().fetchAll();
            return resources;
        } catch (error) {
            console.log('Error fetching from Cosmos DB:', error.message);
        }
    }
    return [
        { id: '1', name: 'Premium Organic Ghee', category: 'ghee', price: 24.99, description: '100% Pure Organic Grass-Fed Ghee', weight: '500ml', inStock: true },
        { id: '2', name: 'Traditional Organic Ghee', category: 'ghee', price: 19.99, description: 'Traditional Hand-Churned Organic Ghee', weight: '250ml', inStock: true }
    ];
}

app.get('/', async (req, res) => {
    const products = await getProducts();
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Organic Ghee Store - New Look!</title>
        <style>
            /* ========== GREEN VERSION - CHANGED CSS COLORS ========== */
            * { margin: 0; padding: 0; box-sizing: border-box; }
            body {
                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                background-color: #e8f5e9;
                color: #1b5e20;
            }
            header {
                background: linear-gradient(135deg, #1b5e20, #4caf50);
                color: white;
                padding: 20px 0;
                text-align: center;
                box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            }
            header h1 {
                font-size: 2.5em;
                margin-bottom: 5px;
            }
            .slot-badge {
                background-color: #4caf50;
                color: white;
                padding: 5px 15px;
                border-radius: 20px;
                display: inline-block;
                margin-top: 10px;
                font-size: 0.9em;
                font-weight: bold;
                border: 2px solid #fff;
            }
            .container {
                max-width: 1200px;
                margin: 30px auto;
                padding: 0 20px;
            }
            .tagline {
                text-align: center;
                font-size: 1.3em;
                color: #2e7d32;
                margin-bottom: 30px;
                font-style: italic;
            }
            .products-grid {
                display: grid;
                grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
                gap: 25px;
                padding: 20px 0;
            }
            .product-card {
                background: white;
                border-radius: 15px;
                padding: 30px;
                box-shadow: 0 4px 15px rgba(27,94,32,0.15);
                transition: transform 0.3s ease;
                border-top: 4px solid #4caf50;
                border-left: 4px solid #4caf50;
            }
            .product-card:hover {
                transform: translateY(-5px);
                box-shadow: 0 8px 25px rgba(27,94,32,0.25);
            }
            .product-card h2 {
                color: #1b5e20;
                margin-bottom: 10px;
                font-size: 1.4em;
            }
            .product-card .price {
                font-size: 1.8em;
                color: #2e7d32;
                font-weight: bold;
                margin: 15px 0;
            }
            .product-card .description {
                color: #555;
                line-height: 1.6;
                margin-bottom: 10px;
            }
            .product-card .weight {
                color: #888;
                font-size: 0.9em;
            }
            .stock-badge {
                display: inline-block;
                padding: 5px 12px;
                border-radius: 12px;
                font-size: 0.85em;
                font-weight: bold;
                margin-top: 10px;
            }
            .in-stock {
                background-color: #c8e6c9;
                color: #1b5e20;
            }
            .buy-btn {
                display: block;
                width: 100%;
                padding: 12px;
                margin-top: 15px;
                background: linear-gradient(135deg, #2e7d32, #4caf50);
                color: white;
                border: none;
                border-radius: 8px;
                font-size: 1.1em;
                cursor: pointer;
                transition: all 0.3s;
            }
            .buy-btn:hover {
                background: linear-gradient(135deg, #1b5e20, #388e3c);
                transform: scale(1.02);
            }
            .new-badge {
                background-color: #ff5722;
                color: white;
                padding: 3px 10px;
                border-radius: 10px;
                font-size: 0.75em;
                margin-left: 10px;
                vertical-align: middle;
            }
            footer {
                text-align: center;
                padding: 20px;
                background: linear-gradient(135deg, #1b5e20, #4caf50);
                color: white;
                margin-top: 50px;
            }
        </style>
    </head>
    <body>
        <header>
            <h1>🧈 Organic Ghee Store <span class="new-badge">NEW LOOK</span></h1>
            <p>Pure. Natural. Traditional.</p>
            <span class="slot-badge">🟢 GREEN Deployment (Staging)</span>
        </header>
        <div class="container">
            <p class="tagline">"From Farm to Table - The Purest Ghee Experience" 🌿</p>
            <div class="products-grid">
                ${products.map(p => `
                    <div class="product-card">
                        <h2>${p.name}</h2>
                        <p class="description">${p.description}</p>
                        <p class="price">$${p.price}</p>
                        <p class="weight">Weight: ${p.weight}</p>
                        <span class="stock-badge in-stock">${p.inStock ? '✅ In Stock' : '❌ Out of Stock'}</span>
                        <button class="buy-btn">🛒 Add to Cart</button>
                    </div>
                `).join('')}
            </div>
        </div>
        <footer>
            <p>&copy; 2024 Organic Ghee Store | Slot: ${slotName.toUpperCase()} | Version 2.0 🌿</p>
        </footer>
    </body>
    </html>
    `);
});

app.get('/health', (req, res) => {
    res.json({ status: 'healthy', slot: slotName, version: '2.0', timestamp: new Date().toISOString() });
});

app.get('/api/products', async (req, res) => {
    const products = await getProducts();
    res.json(products);
});

initCosmosDB().then(() => {
    app.listen(port, () => {
        console.log(`Organic Ghee Store (${slotName}) running on port ${port}`);
    });
});
APPEOF

# Create zip file
zip -r organicghee-green.zip .

# Deploy to the GREEN slot
az webapp deployment source config-zip \
  --resource-group rg-organicghee-prod \
  --name app-organicghee-12345 \
  --slot green \
  --src organicghee-green.zip

echo "Green deployment completed!"
```

### Step 15: Verify Green Deployment
1. Go back to Azure Portal
2. Navigate to your App Service → **"Deployment slots"**
3. Click on the **green** slot
4. Click the **Default domain** URL (e.g., `https://app-organicghee-12345-green.azurewebsites.net`)
5. You should see the **Green-themed** Organic Ghee Store with:
   - 🟢 Green header (gradient green instead of brown)
   - Green card borders
   - Green buttons
   - "NEW LOOK" badge
   - 🟢 GREEN Deployment badge

---

## PHASE 5: WAF POLICY CREATION

### Step 16: Create WAF Policy
1. Search **"Web Application Firewall policies"** in the search bar
2. Click **"+ Create"**
3. **Basics tab**:
   - **Policy for**: `Application Gateway`
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Policy name**: `waf-organicghee`
   - **Location**: `East US`
4. **Policy Settings tab**:
   - **Mode**: `Prevention`
   - **Max request body size (KB)**: `128`
   - **Max file upload size (MB)**: `100`
   - Check **"Inspect request body"**: ✅
5. **Managed rules tab**:
   - **Managed rule set**: Leave default `OWASP 3.2` selected
   - Click **"Next"**
6. **Custom rules tab**: Leave empty → Click **"Next"**
7. **Association tab**: Leave empty for now (we'll associate with App Gateway later)
8. Click **"Review + create"**
9. Click **"Create"**

---

## PHASE 6: APPLICATION GATEWAY SETUP

### Step 17: Create Public IP for Application Gateway
1. Search **"Public IP addresses"** in the search bar
2. Click **"+ Create"**
3. **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Region**: `East US`
   - **Name**: `pip-appgw-organicghee`
   - **IP Version**: `IPv4`
   - **SKU**: `Standard`
   - **Tier**: `Regional`
   - **Assignment**: `Static`
4. Click **"Review + create"**
5. Click **"Create"**

### Step 18: Create Application Gateway
1. Search **"Application gateways"** in the search bar
2. Click **"+ Create"**

#### **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Application gateway name**: `appgw-organicghee`
   - **Region**: `East US`
   - **Tier**: `WAF V2`
   - **Enable autoscaling**: `Yes`
   - **Minimum scale units**: `0`
   - **Maximum scale units**: `2`
   - **Availability zone**: `None`
   - **HTTP2**: `Enabled`
   - **WAF Policy**: Select `waf-organicghee` (the one we created)
   - **Virtual network**: Select `vnet-organicghee`
   - **Subnet**: Select `subnet-appgw` (10.0.1.0/24)
3. Click **"Next: Frontends"**

#### **Frontends tab**:
   - **Frontend IP address type**: `Public`
   - **Public IP address**: Select `pip-appgw-organicghee`
4. Click **"Next: Backends"**

#### **Backends tab**:
5. Click **"+ Add a backend pool"**:
   - **Name**: `bp-blue`
   - **Add backend pool without targets**: `No`
   - Click **"Add a target"**:
     - **Target type**: `App Services`
     - **Target**: Select `app-organicghee-12345` (production slot)
   - Click **"Add"**
6. Click **"+ Add a backend pool"** again:
   - **Name**: `bp-green`
   - **Add backend pool without targets**: `No`
   - Click **"Add a target"**:
     - **Target type**: `App Services`
     - **Target**: Select `app-organicghee-12345-green` (green slot)
     
     > **Note**: If the green slot doesn't appear, select **App Services** and look for the green slot name. If it still doesn't appear, you can add it as an **IP address or FQDN**:
     > - **Target type**: `IP address or FQDN`
     > - **Target**: `app-organicghee-12345-green.azurewebsites.net`
   - Click **"Add"**
7. Click **"Next: Configuration"**

#### **Configuration tab**:
8. Click **"+ Add a routing rule"**:

   **Listener tab**:
   - **Rule name**: `rule-blue`
   - **Priority**: `100`
   - **Listener name**: `listener-http`
   - **Frontend IP**: `Public`
   - **Protocol**: `HTTP`
   - **Port**: `80`
   - **Listener type**: `Basic`
   - **Error page url**: `No`

   **Backend targets tab**:
   - **Target type**: `Backend pool`
   - **Backend target**: `bp-blue`
   - **Backend settings**: Click **"Add new"**:
     - **Backend settings name**: `settings-appservice`
     - **Backend protocol**: `HTTPS`
     - **Backend port**: `443`
     - **Cookie-based affinity**: `Disable`
     - **Connection draining**: `Disable`
     - **Request time-out (seconds)**: `30`
     - **Override backend path**: Leave empty
     - **Override with new host name**: `Yes`
     - **Host name override**: Select **"Pick host name from backend target"**
     - Click **"Add"**
   - Click **"Add"** (for the routing rule)

9. Click **"Next: Tags"** → Leave empty
10. Click **"Next: Review + create"**
11. Click **"Create"**

> ⚠️ **The Application Gateway deployment takes 15-25 minutes. Please wait.**

### Step 19: Verify Application Gateway
1. Once deployed, click **"Go to resource"**
2. In **"Overview"**, find the **"Frontend public IP address"**
3. Copy the IP address
4. Open a browser and navigate to: `http://<APP-GW-IP-ADDRESS>`
5. You should see the **Blue version** of the Organic Ghee Store

### Step 20: Add Routing Rule for Green Slot (Path-based)
1. In the Application Gateway, click **"Rules"** in the left menu under **Settings**
2. Click on your existing rule `rule-blue`
3. **OR** we'll add a path-based rule to access green:

**Alternative: Access green via a separate rule**
1. Click **"Listeners"** in the left menu
2. Click **"+ Add listener"**:
   - **Listener name**: `listener-green`
   - **Frontend IP**: `Public`
   - **Port**: `8080`
   - **Protocol**: `HTTP`
   - Click **"Add"**
3. Click **"Rules"** → Click **"+ Routing rule"**:
   - **Rule name**: `rule-green`
   - **Priority**: `200`
   - **Listener**: `listener-green`
   - **Backend targets**:
     - **Target type**: `Backend pool`
     - **Backend target**: `bp-green`
     - **Backend settings**: `settings-appservice`
   - Click **"Add"**

> **Note**: If port 8080 doesn't work with your NSG, you can instead use path-based routing. For now, port 80 routes to Blue, and we'll use Front Door to manage routing.

### Step 21: Add NSG rule for Port 8080 (if using separate port for green)
1. Search **"Network security groups"** in the search bar
2. If an NSG is associated with `subnet-appgw`, click on it
3. Click **"Inbound security rules"** → **"+ Add"**:
   - **Source**: `Any`
   - **Source port ranges**: `*`
   - **Destination**: `Any`
   - **Destination port ranges**: `8080`
   - **Protocol**: `TCP`
   - **Action**: `Allow`
   - **Priority**: `110`
   - **Name**: `Allow-8080`
4. Click **"Add"**

---

## PHASE 7: AZURE FRONT DOOR SETUP

### Step 22: Create Azure Front Door
1. Search **"Front Door and CDN profiles"** in the search bar
2. Click **"+ Create"**
3. Select **"Azure Front Door"** (not Classic)
4. Click **"Continue to create a Front Door"**
5. Click **"Custom create"** tab

#### **Basics tab**:
   - **Subscription**: Select your subscription
   - **Resource group**: `rg-organicghee-prod`
   - **Name**: `fd-organicghee` (must be globally unique, e.g., `fd-organicghee-12345`)
   - **Tier**: `Standard` (or Premium for more WAF features)
6. Click **"Next: Endpoint"**

#### **Endpoint tab**:
7. Click **"+ Add an endpoint"**:
   - **Endpoint name**: `ep-organicghee`
   - **Status**: `Enabled`
   - Click **"Add"**

8. Click **"Next: Route"**

#### **Route tab**:
9. Click **"+ Add a route"**:
   - **Name**: `route-to-appgw`
   - **Endpoint**: `ep-organicghee`
   - **Domains**: Select the auto-generated domain (e.g., `ep-organicghee-xxxxxxx.z01.azurefd.net`)
   - **Patterns to match**: `/*`
   - **Accepted protocols**: `HTTP and HTTPS`
   - **Redirect**: `Disable`
   - **Origin group**: Click **"Add a new origin group"**:
     - **Name**: `og-appgw`
     - Click **"+ Add an origin"**:
       - **Name**: `origin-appgw`
       - **Origin type**: `Custom`
       - **Host name**: Enter the **Application Gateway Public IP address** (e.g., `20.xxx.xxx.xxx`)
       - **Origin host header**: `app-organicghee-12345.azurewebsites.net`
       - **HTTP port**: `80`
       - **HTTPS port**: `443`
       - **Priority**: `1`
       - **Weight**: `1000`
       - **Status**: `Enable this origin`
       - Click **"Add"**
     - **Health probes**:
       - **Health probe path**: `/health`
       - **Protocol**: `HTTP`
       - **Probe method**: `GET`
       - **Interval (seconds)**: `30`
     - **Load balancing**:
       - **Sample size**: `4`
       - **Successful samples required**: `3`
       - **Latency sensitivity (ms)**: `0`
     - Click **"Add"** (for origin group)
   - **Forwarding protocol**: `HTTP only` (since App GW is on HTTP currently)
   - **Caching**: `Disable` (for testing)
   - Click **"Add"** (for route)

10. Click **"Next: Security"** → Leave defaults (or add a Front Door WAF if using Premium tier)
11. Click **"Review + create"**
12. Click **"Create"**
13. **Wait for deployment** (takes 5-10 minutes)

### Step 23: Configure Application Gateway to Accept Front Door Traffic
1. Go to the **Application Gateway** resource
2. Note: Front Door sends traffic with `X-Azure-FDID` header. For production, you should restrict traffic to only Front Door.

**Add Front Door ID to App Gateway (optional but recommended):**
1. Go to the **Front Door** resource
2. Click **"Overview"**
3. Copy the **Front Door ID** (a GUID)
4. This can be used with custom rules in WAF to only accept traffic from your specific Front Door

### Step 24: Verify Front Door
1. Go to the Front Door resource
2. In **"Overview"**, find the **"Endpoint hostname"** (e.g., `ep-organicghee-xxxxxxx.z01.azurefd.net`)
3. Open a browser and navigate to: `https://ep-organicghee-xxxxxxx.z01.azurefd.net`
4. You should see the **Blue version** of the Organic Ghee Store
5. The traffic flow is now: **Front Door → Application Gateway → App Service (Blue)**

---

## PHASE 8: RESTRICT APP SERVICE ACCESS

### Step 25: Restrict App Service to Only Accept Traffic from Application Gateway
1. Go to the **App Service** (`app-organicghee-12345`)
2. Click **"Networking"** in the left menu
3. Under **"Inbound traffic"**, click **"Access restriction"**
4. Under **Unmatched rule action**: Set to **Deny**
5. Click **"+ Add"**:
   - **Name**: `AllowAppGW`
   - **Action**: `Allow`
   - **Priority**: `100`
   - **Type**: `IP Address`
   - **IP Address Block**: Enter the **Application Gateway's Public IP** followed by `/32` (e.g., `20.xxx.xxx.xxx/32`)
6. Click **"Add rule"**

> **Note**: Do the same for the green slot if needed.

---

## PHASE 9: BLUE-GREEN SWAP

### Step 26: Perform Blue-Green Swap (When Ready)
When you're satisfied that the green deployment works correctly:

1. Go to the **App Service** (`app-organicghee-12345`)
2. Click **"Deployment slots"** in the left menu
3. Click **"Swap"** at the top
4. **Swap settings**:
   - **Source**: `green`
   - **Target**: `production` (which is blue)
5. Review the **Config Changes** preview:
   - You'll see that `SLOT_NAME` is marked as "Slot Setting" and will NOT be swapped (stays sticky)
6. Click **"Swap"**
7. Wait for the swap to complete (usually 1-2 minutes)

### Step 27: Verify Swap
1. Navigate to the **production URL**: `https://app-organicghee-12345.azurewebsites.net`
   - Should now show the **GREEN-themed** site (but with "blue" slot name because of sticky setting)
2. Navigate to the **green slot URL**: `https://app-organicghee-12345-green.azurewebsites.net`
   - Should now show the **original BROWN-themed** site
3. Navigate to the **Front Door URL**: `https://ep-organicghee-xxxxxxx.z01.azurefd.net`
   - Should show the **GREEN-themed** site (new production)

---

## PHASE 10: VERIFICATION AND TESTING

### Step 28: Test the Complete Flow
1. **Front Door URL**: `https://ep-organicghee-xxxxxxx.z01.azurefd.net`
   - Verify the page loads
   - Verify products are displayed (from Cosmos DB)

2. **Application Gateway URL**: `http://<APP-GW-IP>`
   - Verify the page loads through App Gateway

3. **Health endpoint**: `https://ep-organicghee-xxxxxxx.z01.azurefd.net/health`
   - Should return JSON with status, slot info

4. **API endpoint**: `https://ep-organicghee-xxxxxxx.z01.azurefd.net/api/products`
   - Should return products from Cosmos DB

### Step 29: Test WAF
1. Go to the **Application Gateway** resource
2. Click **"Web application firewall"** in the left menu
3. Verify WAF is in **"Prevention"** mode
4. Test WAF by trying an SQL injection attack:
   - Navigate to: `http://<APP-GW-IP>/?id=1%20OR%201=1`
   - You should get a **403 Forbidden** error (WAF blocked it)

### Step 30: Monitor WAF Logs
1. In the Application Gateway, click **"Diagnostic settings"** under **Monitoring**
2. Click **"+ Add diagnostic setting"**:
   - **Name**: `appgw-diagnostics`
   - Check: ✅ `ApplicationGatewayAccessLog`
   - Check: ✅ `ApplicationGatewayFirewallLog`
   - **Destination**: Select **"Send to Log Analytics workspace"**
   - Click **"Create new"** workspace if needed:
     - **Name**: `law-organicghee`
     - **Region**: `East US`
     - Click **"OK"**
   - Click **"Save"**

---

## FINAL ARCHITECTURE SUMMARY

```
Internet Users
       ↓
Azure Front Door (fd-organicghee)
  Endpoint: ep-organicghee-xxxxxxx.z01.azurefd.net
       ↓
Application Gateway + WAF V2 (appgw-organicghee)
  WAF Policy: waf-organicghee (Prevention Mode, OWASP 3.2)
  Public IP: pip-appgw-organicghee
       ↓
App Service (app-organicghee-12345)
  ├── Production Slot (Blue): Brown/Gold Theme
  │   SLOT_NAME=blue (sticky)
  └── Green Slot: Green Theme (new CSS)
      SLOT_NAME=green (sticky)
       ↓
Cosmos DB (cosmos-organicghee)
  Database: OrganicGheeDB
  Container: Products
  Partition Key: /category
```

## Resource Summary Table

| Resource | Name | SKU/Tier |
|----------|------|----------|
| Resource Group | rg-organicghee-prod | N/A |
| VNet | vnet-organicghee | N/A |
| Cosmos DB | cosmos-organicghee-12345 | Serverless |
| App Service Plan | asp-organicghee | Standard S1 |
| App Service | app-organicghee-12345 | Standard S1 |
| App Service Slot | green | (shared with plan) |
| WAF Policy | waf-organicghee | WAF V2 |
| Application Gateway | appgw-organicghee | WAF V2 |
| Public IP | pip-appgw-organicghee | Standard |
| Front Door | fd-organicghee | Standard |
| App Insights | ai-organicghee | N/A |
| Log Analytics | law-organicghee | N/A |

---

## COST OPTIMIZATION TIPS

1. **Delete resources** when not in use (especially Application Gateway WAF V2 - ~$350/month)
2. Use **Cosmos DB Serverless** for development
3. Use **App Service Free/Basic** tier if you don't need deployment slots (Standard S1 minimum for slots)
4. **Front Door Standard** is cheaper than Premium

## CLEANUP (When Done)

```bash
# Delete everything at once by deleting the resource group
az group delete --name rg-organicghee-prod --yes --no-wait
```

Or in the portal:
1. Search **"Resource groups"**
2. Click `rg-organicghee-prod`
3. Click **"Delete resource group"**
4. Type the resource group name to confirm
5. Click **"Delete"**

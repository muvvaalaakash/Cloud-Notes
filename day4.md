exposed application using public address but the application is in private vm

✅ Complete Steps We Followed - Clear Summary
🏗️ Architecture
text

Internet (Port 80)
       |
       ↓
+------------------+
|   Public VM      |
|   (Jump Server)  |
|   Nginx Only     |
|   Public IP ✅   |
+------------------+
       |
       | Private Network
       ↓
+------------------+
|   Private VM     |
|   Node.js App    |
|   Port 5656      |
|   No Public IP   |
+------------------+
📋 STEP 1: Connect to Public VM
Bash

# From your local machine
ssh -i <key.pem> azureuser@<PUBLIC_VM_IP>
📋 STEP 2: Connect to Private VM from Public VM
Bash

# From Public VM
ssh azureuser@10.0.2.4
📋 STEP 3: Setup on Private VM
Install Node.js
Bash

sudo apt update
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y
node -v
npm -v
Clone the Repository
Bash

git clone <your-repo-url>
cd organic-ghee
Install Dependencies
Bash

npm install
Start the App
Bash

npm start
# App runs on port 5656
Verify App is Running
Bash

curl http://localhost:5656
# Should return your HTML ✅
📋 STEP 4: Go Back to Public VM
Bash

exit
# Now you are back on Public VM
Verify Private VM App is Reachable
Bash

curl http://10.0.2.4:5656
# Should return your HTML ✅
📋 STEP 5: Setup Nginx on Public VM
Install Nginx
Bash

sudo apt update
sudo apt install nginx -y
Configure Nginx
Bash

sudo nano /etc/nginx/sites-available/default
Paste This Config
nginx

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    location / {
        proxy_pass http://10.0.2.4:5656;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';

        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
Save: CTRL + X → Y → Enter

Test and Restart Nginx
Bash

sudo nginx -t
sudo systemctl restart nginx
sudo systemctl status nginx
📋 STEP 6: Open Port 80 on Public VM
Bash

sudo ufw allow 80
sudo ufw allow 443
sudo ufw reload
📋 STEP 7: Access Application
Open browser and go to:

text

http://<PUBLIC_VM_PUBLIC_IP>
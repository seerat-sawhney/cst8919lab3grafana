# Grafana Installation and Dashboard Setup on Ubuntu Server in Azure



## Setup Process

### Step 1: Create an Ubuntu VM in Azure
1. **Log in to Azure Portal** and navigate to **Virtual Machines**.
2. Click **Create > Azure Virtual Machine**.
3. Fill in the details:
   - **Subscription:** Select your subscription.
   - **Resource Group:** Create a new one or use an existing one.
   - **Virtual Machine Name:** Ubuntu-Grafana
   - **Region:** Choose a nearby region.
   - **Image:** Select Ubuntu 20.04 LTS.
   - **Size:** Choose a size like Standard B1s or B2s.
   - **Authentication Type:** SSH public key or Password.
   - **Username:** adminuser.
   - **Networking:** Ensure SSH (22) is allowed.
4. Click **Review + Create**, then **Create**.


        ![alt text](s1.png)
   ![alt text](images/s1.png)

   
### Step 2: Connect to Your Ubuntu VM
1. Go to **Virtual Machines** and select your Ubuntu-Grafana VM.
2. Copy the **public IP address**.
3. Open Terminal and connect via SSH:
   ```sh
   ssh adminuser@<your-vm-public-ip>
   ```
4. Enter the password if prompted.

### Step 3: Prepare Ubuntu Server
1. Update system packages:
   ```sh
   sudo apt-get update && sudo apt-get upgrade -y
   ```
2. Install dependencies:
   ```sh
   sudo apt-get install -y apt-transport-https software-properties-common wget
   ```

### Step 4: Install Grafana
1. Add Grafana’s official APT repository:
   ```sh
   sudo mkdir -p /etc/apt/keyrings/
   wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
   echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
   ```
2. Install Grafana:
   ```sh
   sudo apt-get update
   sudo apt-get install grafana -y
   ```
3. Start and enable Grafana service:
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```
4. Verify Grafana is running:
   ```sh
   sudo systemctl status grafana-server
   ```

### Step 5: Allow Port 3000 for Grafana
```sh
sudo ufw allow 3000/tcp
sudo ufw status
```

### Step 6: Connect Grafana to Azure Monitor
1. **Enable Managed Identity for the VM:**
   - Go to **Azure Portal > Virtual Machines > Ubuntu-Grafana**.
   - Click **Identity** > **System assigned** > **On** > **Save**.
2. **Assign Permissions:**
   - Go to **Azure Monitor > Access Control (IAM)**.
   - Add **Monitoring Reader** role to VM’s Managed Identity.
   - Go to **Subscriptions > IAM > Add role assignment**.
   - Assign **Reader Role** to the same Managed Identity.
3. **Configure Managed Identity in Grafana:**
   ```sh
   sudo nano /etc/grafana/grafana.ini
   ```
   Edit the following:
   ```ini
   [auth.azure]
   enabled = true
   managed_identity_enabled = true
   ```
   Save and restart Grafana:
   ```sh
   sudo systemctl restart grafana-server
   ```

### Step 7: Access Grafana and Add Data Source
1. Open **http://<your-vm-public-ip>:3000**.
2. Log in with:
   - **Username:** admin
   - **Password:** admin (change it after first login).
3. Add **Azure Monitor** as Data Source:
   - Click **Configuration > Data Sources > Add data source**.
   - Select **Azure Monitor**.
   - In **Authentication**, choose **Managed Identity**.
   - Click **Save & Test**.

### Step 8: Create a Dashboard
1. Click **+** on the left sidebar > **Dashboard**.
2. Click **Add new panel**.
3. Select **Azure Monitor** as the data source.
4. Choose metrics like:
   - CPU usage
   - Memory usage
   - Network I/O
5. Customize visualization and click **Apply**, then **Save Dashboard**.

---

## Issues Encountered
### Problem: Entra ID Not Working
- **Issue:** Unable to configure the Managed Identity due to Entra ID issues in Azure.
- **Solution Attempted:** Verified IAM role assignments and tried enabling the identity again.
- **Current Status:** Data source cannot fetch metrics due to Entra ID issue.

### Problem: Initial Inability to Access Grafana Interface
- **Issue:** Could not access Grafana on port 3000 initially.
- **Solution:** Added an inbound security rule for port 3000 in Azure VM networking settings.
- 
---

## Submission
1. **Screenshots:**
   - Grafana installation.
   - Grafana running (`sudo systemctl status grafana-server`).
   - Azure Monitor data source setup.
   - Final dashboard with performance metrics.
2. **Report (.MD file):**
   - Steps taken.
   - Issues faced and solutions attempted.
3. **GitHub Submission:**
   - Create a repository.
   - Upload screenshots and `README.md`.
   - Submit the GitHub link.

**Note:** Due to Entra ID issues, the data source is not fully functional. Further troubleshooting on Azure IAM permissions may be required.


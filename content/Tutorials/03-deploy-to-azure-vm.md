+++
title = "3. Deploy Spring Boot App to Azure VM (Simple)"
weight = 3
date = 2025-11-11
draft = false
tags = ["week1", "JIN"]
+++

# Deploy Spring Boot App to Azure VM (Simple)

## Goal

Deploy your Spring Boot application to an Azure Virtual Machine using the simplest manual approach to understand the fundamentals of cloud deployment.

> **What you'll learn:**
>
> - How to provision an Azure Ubuntu VM
> - When to use Virtual Machines for Java applications
> - Best practices for basic cloud deployments

## Prerequisites

> **Before starting, ensure you have:**
>
> - ✓ Completed "Build Your First Spring Boot Web Application" tutorial
> - ✓ Azure account (free tier works: <https://azure.microsoft.com/free/>)
> - ✓ Azure CLI installed (<https://docs.microsoft.com/cli/azure/install-azure-cli>)
> - ✓ SSH key pair generated (`ssh-keygen -t rsa -b 4096`)

## Exercise Steps

### Overview

1. **Create Azure Virtual Machine**
2. **Configure VM and Install Java**
3. **Build Application Locally**
4. **Copy Application to VM**
5. **Run Application on VM**
6. **Configure Azure Firewall**
7. **Test Your Deployment**

### **Step 1:** Create Azure Virtual Machine

Provision an Ubuntu Linux virtual machine in Azure to host your Spring Boot application. Virtual machines provide full control over the operating system and are a fundamental cloud computing service.

1. **Log in to Azure Portal** at <https://portal.azure.com>

2. **Navigate to Virtual Machines**
   - Click "Create" → "Azure virtual machine"

3. **Configure Basic Settings:**

   **Project details:**
   - Subscription: Select your subscription
   - Resource group: Click "Create new" → name it `hellojava-dev-rg`

   **Instance details:**
   - Virtual machine name: `hellojava-dev-vm`
   - Region: `North Europe`
   - Availability options: No infrastructure redundancy required
   - Security type: Standard
   - Image: `Ubuntu Server 24.04 LTS - x64 Gen2`
   - Size: `Standard_B1s` (1 vCPU, 1 GB RAM) - Click "See all sizes" if not visible

   **Administrator account:**
   - Authentication type: SSH public key
   - Username: `azureuser`
   - SSH public key source: Use existing public key
   - SSH public key: Paste your public key from `~/.ssh/id_rsa.pub`

4. **Configure Inbound Port Rules:**
   - Public inbound ports: Allow selected ports
   - Select inbound ports: SSH (22)
   - Note: We'll add port 8080 later

5. **Review + Create**
   - Click "Review + create"
   - Wait for validation
   - Click "Create"
   - Wait 2-3 minutes for deployment

6. **Note the Public IP Address**
   - Once deployed, click "Go to resource"
   - Copy the **Public IP address** (you'll need this)

> ℹ **Concept Deep Dive**
>
> Azure Virtual Machines are IaaS (Infrastructure as a Service) offerings that provide full control over the operating system. The B1s size (1 vCPU, 1 GB RAM) costs approximately $10/month and is sufficient for development and testing. Ubuntu Server 24.04 LTS (Long Term Support) receives security updates for 5 years, making it ideal for production workloads. SSH key authentication is more secure than passwords - the private key stays on your machine while the public key is stored in Azure.
>
> ⚠ **Common Mistakes**
>
> - Choosing a larger VM size increases costs significantly - B1s is perfect for this tutorial
> - Not copying the public IP address means you'll have to find it later in the portal
> - Using password authentication instead of SSH keys creates security vulnerabilities
> - Forgetting which resource group you created makes cleanup harder later
>
> ✓ **Quick check:** VM status shows "Running" and you have the public IP address copied

### **Step 2:** Configure VM and Install Java

Connect to your VM via SSH and install Java 21, which is required to run your Spring Boot application.

1. **Connect to your VM via SSH:**

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   ```

   Replace `<YOUR_PUBLIC_IP>` with the IP address you copied. Type "yes" when prompted about authenticity.

2. **Update package lists:**

   ```bash
   sudo apt update
   ```

3. **Install Java 21:**

   ```bash
   sudo apt install -y openjdk-21-jre-headless
   ```

   The `-headless` variant excludes GUI libraries (not needed for servers).

4. **Verify Java installation:**

   ```bash
   java -version
   ```

   Expected output: `openjdk version "21.0.x"`

> ℹ **Concept Deep Dive**
>
> The JRE (Java Runtime Environment) is sufficient for running Java applications - we don't need the full JDK (Java Development Kit) on the server since we're not compiling code there. The `openjdk-21-jre-headless` package is optimized for servers without GUI requirements, saving disk space and memory. Running `apt update` before installation ensures you get the latest package versions with security patches.
>
> ⚠ **Common Mistakes**
>
> - Installing the JDK instead of JRE wastes resources on a production server
> - Skipping `apt update` might install outdated packages with security vulnerabilities
> - Installing Java 17 or older won't work with Spring Boot 3.5.7 (requires Java 17+)
>
> ✓ **Quick check:** `java -version` shows version 21.0.x

### **Step 3:** Build Application Locally

Build your Spring Boot application on your local machine to create the deployable JAR file.

1. **Open a new terminal on your local machine** (or use your existing terminal if you've exited SSH)

2. **Navigate to your project directory:**

   ```bash
   cd HelloJava
   ```

   Make sure you're in the project root directory

3. **Build the application:**

   ```bash
   ./mvnw clean package
   ```

4. **Verify the JAR file was created:**

   ```bash
   ls -lh target/*.jar
   ```

   You should see `hellojava-0.0.1-SNAPSHOT.jar` (approximately 20-25 MB)

> ℹ **Concept Deep Dive**
>
> The `mvn clean package` command removes old build artifacts (`clean`) and creates a new executable JAR file (`package`). Spring Boot's Maven plugin creates a "fat JAR" or "uber JAR" - a single file containing your application code, all dependencies, and an embedded Tomcat server. This makes deployment simple: one file contains everything needed to run the application. The SNAPSHOT suffix indicates this is a development version, not a formal release.
>
> ⚠ **Common Mistakes**
>
> - Not running `clean` first can leave old classes in the JAR, causing confusing behavior
> - Forgetting to check that the JAR exists before trying to copy it
> - Accidentally building with `-DskipTests` and missing test failures
>
> ✓ **Quick check:** JAR file exists at `target/hellojava-0.0.1-SNAPSHOT.jar` and is 20-25 MB

### **Step 4:** Copy Application to VM

Transfer the JAR file from your local machine to the Azure VM using SCP (Secure Copy Protocol).

1. **From your local terminal, copy the JAR file:**

   ```bash
   scp target/hellojava-0.0.1-SNAPSHOT.jar azureuser@<YOUR_PUBLIC_IP>:~/
   ```

   Replace `<YOUR_PUBLIC_IP>` with your VM's public IP address.

2. **Wait for the transfer to complete** (should take 10-30 seconds depending on your internet speed)

3. **Verify the file arrived on the VM** - in your SSH session to the VM:

   ```bash
   ls -lh ~/hellojava-0.0.1-SNAPSHOT.jar
   ```

   You should see the JAR file in the home directory.

> ℹ **Concept Deep Dive**
>
> SCP (Secure Copy Protocol) transfers files over SSH, providing encrypted transfer. The syntax is `scp source destination` where the destination can be a remote location in the format `user@host:path`. The `~` symbol represents the user's home directory (`/home/azureuser` in this case). SCP uses the same authentication as SSH, so your SSH key automatically works for file transfers.
>
> ⚠ **Common Mistakes**
>
> - Forgetting the `:~/` at the end results in an error about the destination
> - Using the wrong IP address means the transfer fails or goes to the wrong machine
> - Not verifying the file arrived before proceeding can cause confusion later
> - Spaces in filenames require quotes: `scp "my file.jar" user@host:~/`
>
> ✓ **Quick check:** JAR file exists on VM at `/home/azureuser/hellojava-0.0.1-SNAPSHOT.jar`

### **Step 5:** Run Application on VM

Start your Spring Boot application on the VM to verify it works in the cloud environment.

1. **In your SSH session to the VM, run the application:**

   ```bash
   java -jar hellojava-0.0.1-SNAPSHOT.jar
   ```

2. **Wait for the application to start** - look for this message:

   ```
   Started HelloJavaApplication in X.XXX seconds
   ```

3. **Leave this terminal running** - the application is now active

4. **Optional: Verify locally on the VM** - if you want to confirm the application is running before opening the firewall:

   Open a new SSH session to the VM:
   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   ```

   Test with curl:
   ```bash
   curl http://localhost:8080/
   curl http://localhost:8080/hello
   ```

   You should see HTML content. This step is optional - you can also skip directly to configuring the firewall and testing from your browser.

> ℹ **Concept Deep Dive**
>
> Running `java -jar` directly starts the application in the foreground - your terminal is attached to the application's output. This is useful for initial testing because you see all logs immediately. The application binds to port 8080 on all network interfaces by default. Testing with `curl localhost:8080` verifies the application works on the VM itself before troubleshooting external access. In a later tutorial, we'll use systemd to run the application as a background service.
>
> ⚠ **Common Mistakes**
>
> - Closing the terminal kills the application - use Ctrl+C to stop it intentionally
> - Port 8080 already in use: another process is using that port (check with `sudo lsof -i :8080`)
> - Application fails to start: check Java version matches requirements (`java -version`)
> - Out of memory errors: B1s has only 1 GB RAM, may need to limit Java heap size
>
> ✓ **Quick check:** Application started successfully and `curl localhost:8080` returns HTML content

### **Step 6:** Configure Azure Firewall

Configure Azure's Network Security Group (NSG) to allow traffic to port 8080 so the application is accessible from the internet.

1. **Go back to Azure Portal** in your web browser

2. **Navigate to your VM:**
   - Search for "hellojava-dev-vm" in the top search bar
   - Click on your VM

3. **Open Networking settings:**
   - In the left menu, click "Networking"
   - Click "Network settings"

4. **Add inbound port rule:**
   - Click "Create port rule" → "Inbound port rule"

   Configure the rule:
   - Source: Any
   - Source port ranges: *
   - Destination: Any
   - Service: Custom
   - Destination port ranges: `8080`
   - Protocol: TCP
   - Action: Allow
   - Priority: 310 (or any available number 100-4096)
   - Name: `Allow-HTTP-8080`
   - Description: `Allow HTTP traffic to Spring Boot application`

5. **Click "Add"** and wait a few seconds for the rule to take effect

> ℹ **Concept Deep Dive**
>
> Network Security Groups (NSGs) are Azure's firewall service. By default, Azure VMs block all inbound traffic except SSH (port 22). NSG rules specify which traffic is allowed or denied based on source, destination, protocol, and port. Priority determines rule order - lower numbers are evaluated first. The rule we created allows TCP traffic from any source to port 8080 on our VM. In production, you'd typically use port 80 or 443 with a reverse proxy like Nginx, but port 8080 is simpler for this tutorial.
>
> ⚠ **Common Mistakes**
>
> - Forgetting to add this rule means the application works locally but not externally
> - Using UDP instead of TCP protocol won't work for HTTP traffic
> - Setting priority that conflicts with existing rules can cause unexpected behavior
> - Allowing all ports (*) instead of just 8080 creates security vulnerabilities
>
> ✓ **Quick check:** NSG rule appears in the list with Status "Succeeded"

### **Step 7:** Test Your Deployment

Verify that your application is accessible from the internet and works correctly in the cloud environment.

1. **Ensure your application is still running** on the VM (check your SSH session from Step 5)

2. **Open your web browser and navigate to:**

   ```
   http://<YOUR_PUBLIC_IP>:8080/
   ```

   Replace `<YOUR_PUBLIC_IP>` with your VM's public IP address.

3. **You should see your HelloJava landing page** with the Bootstrap hero section and "Hello" button

4. **Click the "Hello" button** or navigate to:

   ```
   http://<YOUR_PUBLIC_IP>:8080/hello
   ```

   You should see "Hello, World!" message

5. **Test with custom name parameter:**

   ```
   http://<YOUR_PUBLIC_IP>:8080/hello?name=Azure
   ```

   You should see "Hello, Azure!" message

6. **Check the application logs** in your SSH terminal - you should see log entries for each request:

   ```
   INFO ... [http-nio-8080-exec-1] ... GET "/hello", parameters={}
   ```

> ✓ **Success indicators:**
>
> - Landing page displays correctly at `http://<ip>:8080/`
> - Hello page works at `http://<ip>:8080/hello`
> - Query parameters work: `/hello?name=Test` shows "Hello, Test!"
> - Application logs show incoming HTTP requests
> - No errors in terminal or browser console

> ✓ **Final verification checklist:**
>
> - ☐ VM is running in Azure portal
> - ☐ Java 21 is installed on the VM
> - ☐ JAR file is copied to VM
> - ☐ Application starts without errors
> - ☐ Port 8080 is open in NSG rules
> - ☐ Application is accessible via public IP in browser
> - ☐ All pages and features work correctly

## Understanding the Limitations

This simple deployment approach works but has several limitations:

**No Persistence:** If you close the SSH terminal, the application stops running.

**No Auto-Restart:** If the application crashes or the VM reboots, the application won't restart automatically.

**Manual Start:** You must SSH to the VM and manually start the application after any interruption.

**No Log Management:** Logs only appear in the terminal and disappear when the application stops.

**Development Version:** The SNAPSHOT version indicates this isn't a proper release.

In **Tutorial 4: Configure Systemd Service**, we'll address all these limitations using systemd services for automatic restarts, proper logging, and production-ready deployment.

## Stopping the Application

When you're done testing:

1. **Stop the application:**
   - In the SSH session running the application, press `Ctrl+C`
   - Wait for "Stopping service [Tomcat]" message

2. **Exit the SSH sessions:**

   ```bash
   exit
   ```

3. **Optional: Stop the VM to avoid charges** (if not continuing immediately):
   - In Azure Portal, go to your VM
   - Click "Stop" button
   - Remember to "Start" it again when needed

## Common Issues

> **If you encounter problems:**
>
> **Cannot SSH to VM:** Check that your SSH key matches the one configured in Azure. Verify the IP address is correct and the VM is running.
>
> **SCP transfer fails:** Verify SSH works first (`ssh azureuser@<ip>`). Check file path on local machine is correct.
>
> **Application won't start:** Check Java version with `java -version` (must be 21+). Verify JAR file isn't corrupted (`ls -lh` should show 20-25 MB).
>
> **Application starts but can't access from browser:** Verify NSG rule is configured correctly for port 8080. Check application is actually running (`curl localhost:8080` from VM).
>
> **"Address already in use" error:** Port 8080 is in use. Check with `sudo lsof -i :8080` and kill the process or use a different port.
>
> **Out of memory errors:** B1s has only 1 GB RAM. Limit Java heap: `java -Xmx512m -jar hellojava-0.0.1-SNAPSHOT.jar`
>
> **Still stuck?** Check application logs in the terminal and VM system logs with `sudo journalctl -n 50`

## Cost Management

> **Important cost considerations:**
>
> - B1s VM costs approximately $10/month when running
> - Stopped (deallocated) VMs only cost ~$1/month for disk storage
> - Public IP addresses may incur small charges
> - Data transfer out of Azure (egress) has costs after free tier
> - **Clean up resources when done:** Delete the entire resource group to avoid ongoing charges

## Summary

You've successfully deployed your Spring Boot application to Azure:

- ✓ Provisioned an Azure Ubuntu VM
- ✓ Installed Java on a cloud server
- ✓ Transferred and ran your application in the cloud
- ✓ Made the application accessible from the internet

> **Key takeaway:** Virtual Machines provide full control and are conceptually simple - you're essentially renting a computer in the cloud. This approach works but requires manual intervention for production concerns like auto-restart, logging, and process management. The next tutorial will transform this into a production-ready deployment using systemd services and proper release versioning.

## Going Deeper (Optional)

> **Want to explore more?**
>
> - Configure application properties for different environments
> - Set up a custom domain name and DNS
> - Install and configure Nginx as a reverse proxy (port 80)
> - Add SSL/TLS certificate with Let's Encrypt
> - Set up application monitoring and alerting
> - Configure automatic security updates on Ubuntu

## Clean Up

To avoid ongoing charges, delete all Azure resources when you're done:

```bash
az group delete --name hellojava-dev-rg --yes --no-wait
```

Or use the Azure Portal:
1. Go to Resource Groups
2. Select `hellojava-dev-rg`
3. Click "Delete resource group"
4. Type the resource group name to confirm
5. Click "Delete"

## Done! 🎉

Excellent work! You've deployed a Spring Boot application to Azure and made it accessible to the world. This is your first step into cloud deployment. In the next tutorial, you'll learn how to make this deployment production-ready with systemd services, proper versioning, and automatic restarts.

## Appendix: Infrastructure as Code

The manual steps in this tutorial help you understand the fundamentals. For repeated deployments, automation scripts are available in the `scripts/03-deploy-to-azure-vm/` directory.

> **Note:** The script examples shown below are for reference. Always use the actual script files in the `scripts/03-deploy-to-azure-vm/` directory for the most up-to-date and tested versions.

**`scripts/03-deploy-to-azure-vm/provision-vm.sh`** - Automates VM creation using Azure CLI:

```bash
#!/bin/bash
# Provision Azure VM with Java pre-installed using cloud-init

RESOURCE_GROUP="hellojava-dev-rg"
VM_NAME="hellojava-dev-vm"
LOCATION="northeurope"
IMAGE="Ubuntu2404"
SIZE="Standard_B1s"
CLOUD_INIT_FILE="cloud-init.yaml"

# Check that you are in the same directory as the cloud-init file
if [ ! -f $CLOUD_INIT_FILE ]; then
  echo "Error: $CLOUD_INIT_FILE not found!"
  exit 1
fi

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create VM with cloud-init
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image $IMAGE \
  --size $SIZE \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data @$CLOUD_INIT_FILE \
  --public-ip-sku Standard

# Open port 8080
az vm open-port \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --port 8080 \
  --priority 310

# Get public IP and output it to the terminal
az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --show-details \
  --query publicIps \
  --output tsv
```

**`scripts/03-deploy-to-azure-vm/cloud-init.yaml`** - Configures the VM during provisioning:

```yaml
#cloud-config
package_update: true
packages:
  - openjdk-21-jre-headless
```

> **Using the automation scripts:**
>
> 1. Ensure Azure CLI is installed and logged in (`az login`)
> 2. Run: `cd scripts/03-deploy-to-azure-vm && bash provision-vm.sh`
> 3. Script outputs the public IP when complete
> 4. Continue from Step 3 (Build Application Locally) in the tutorial

> **When to use automation:**
>
> - Creating multiple VMs for testing
> - CI/CD pipelines
> - Disaster recovery scenarios
> - Team environments where everyone needs identical setups

### Build and Deploy Script

For automating Steps 3 and 4 (building and copying the JAR), use this script:

**`scripts/03-deploy-to-azure-vm/deploy-app.sh`** - Automates building and copying:

```bash
#!/bin/bash
# Build and deploy script for Tutorial 3 (Simple VM Deployment)
# Usage: ./deploy-app.sh <VM_PUBLIC_IP>

VM_IP="$1"
SNAPSHOT_JAR="target/hellojava-0.0.1-SNAPSHOT.jar"

# Check if VM IP is provided
if [ -z "$VM_IP" ]; then
  echo "Usage: ./$0 <VM_PUBLIC_IP>"
  exit 1
fi

# Check if Maven wrapper exists
if [ ! -f ./mvnw ]; then
  echo "Error: Maven wrapper (mvnw) not found!"
  # Inform the user to run from the project root
  echo "Please run this script from the project root directory."
  exit 1
fi

echo "Building application..."
./mvnw clean package

echo "Copying JAR to VM..."
scp $SNAPSHOT_JAR azureuser@${VM_IP}:~/

echo "Done! SSH to the VM and run: java -jar $SNAPSHOT_JAR"
```

**Using the script:**

```bash
# First provision the VM (Steps 1-2)
cd scripts/03-deploy-to-azure-vm && bash provision-vm.sh
# Note the public IP
cd ../..

# Then build and copy (Steps 3-4)
bash scripts/03-deploy-to-azure-vm/deploy-app.sh <VM_PUBLIC_IP>

# Continue with Steps 5-7 manually
```

This script only automates Steps 3 and 4. You still provision the VM manually (Steps 1-2) and start the application manually (Steps 5-7) to understand those concepts.

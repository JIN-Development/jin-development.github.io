+++
title = "4. Deploy Spring Boot App to Azure VM with Systemd"
weight = 4
date = 2025-11-11
draft = false
tags = ["week1", "JIN"]
+++

# Deploy Spring Boot App to Azure VM with Systemd

## Goal

Deploy your Spring Boot application to Azure as a production-ready service with automatic startup using systemd process management.

> **What you'll learn:**
>
> - How to create dedicated system users for running applications
> - When to use systemd for application management
> - Best practices for production deployments on Linux
> - How to configure automatic restart and boot behavior

## Prerequisites

> **Before starting, ensure you have:**
>
> - ✓ Completed Tutorial 3: "Deploy to Azure VM"
> - ✓ Understanding of basic Linux commands and file permissions
> - ✓ A working Spring Boot application (SNAPSHOT version is fine)

## Exercise Steps

### Overview

1. **Provision New Ubuntu VM**
2. **Configure VM for Production**
3. **Deploy Application**
4. **Create systemd Service**
5. **Configure Firewall**
6. **Test and Verify Production Deployment**

### **Step 1:** Provision New Ubuntu VM

Create a fresh VM for the production deployment following the same steps as the simple tutorial.

1. **Log in to Azure Portal** at <https://portal.azure.com>

2. **Create a new Virtual Machine:**
   - Click "Create" → "Azure virtual machine"

3. **Configure Basic Settings:**

   **Project details:**
   - Resource group: Create new → `hellojava-prod-rg`

   **Instance details:**
   - Virtual machine name: `hellojava-prod-vm`
   - Region: `North Europe`
   - Image: `Ubuntu Server 24.04 LTS - x64 Gen2`
   - Size: `Standard_B1s`

   **Administrator account:**
   - Authentication type: SSH public key
   - Username: `azureuser`
   - SSH public key: Paste from `~/.ssh/id_rsa.pub`

4. **Inbound ports:** SSH (22) only for now

5. **Review + Create** → wait for deployment

6. **Copy the Public IP address**

> ℹ **Concept Deep Dive**
>
> Creating a separate resource group (`hellojava-prod-rg`) keeps production resources isolated from development/testing resources. This makes it easier to manage, monitor, and delete resources independently. In real-world scenarios, you'd also use different Azure subscriptions or management groups to separate environments. The "prod" naming convention clearly identifies production resources and helps prevent accidental modifications.
>
> ✓ **Quick check:** VM is running and you have the public IP address

### **Step 2:** Configure VM for Production

Set up the VM with proper Linux user accounts, directory structure, and permissions for running production applications.

1. **Connect to your VM via SSH:**

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   ```

2. **Update system packages:**

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install Java 21 JRE:**

   ```bash
   sudo apt install -y openjdk-21-jre-headless
   ```

4. **Verify Java installation:**

   ```bash
   java -version
   ```

5. **Create a dedicated application user:**

   ```bash
   sudo useradd -r -s /bin/false hellojava
   ```

   - `-r` creates a system user (no home directory by default)
   - `-s /bin/false` prevents interactive login (security best practice)

6. **Create application directory:**

   ```bash
   sudo mkdir -p /opt/hellojava
   ```

7. **Set ownership:**

   ```bash
   sudo chown hellojava:hellojava /opt/hellojava
   ```

8. **Verify the directory structure:**

   ```bash
   ls -ld /opt/hellojava
   ```

   Should show: `drwxr-xr-x 2 hellojava hellojava`

> ℹ **Concept Deep Dive**
>
> Running applications as dedicated system users (not root or regular users) is a security best practice that limits damage if the application is compromised. The `/opt` directory is the standard Linux location for optional/add-on software installed by administrators. System users with `/bin/false` as their shell cannot log in interactively, reducing the attack surface. Directory ownership ensures the application process can read its files but other users cannot modify them without sudo privileges.
>
> ⚠ **Common Mistakes**
>
> - Running applications as root grants unnecessary privileges and creates security vulnerabilities
> - Creating regular users instead of system users (`-r` flag) clutters the user list
> - Wrong permissions (like 777) allow any user to modify application files
> - Placing applications in `/home` or `/tmp` doesn't follow Linux conventions
>
> ✓ **Quick check:** `/opt/hellojava` directory exists and is owned by `hellojava:hellojava` user

### **Step 3:** Deploy Application

Copy the SNAPSHOT JAR to the VM and set up proper file permissions for production.

1. **From your local machine, build the application:**

   ```bash
   ./mvnw clean package
   ```

2. **Verify the JAR was created:**

   ```bash
   ls -lh target/hellojava-*.jar
   ```

   You should see `hellojava-0.0.1-SNAPSHOT.jar` (or similar version)

3. **Copy the JAR to the VM:**

   ```bash
   scp target/hellojava-0.0.1-SNAPSHOT.jar azureuser@<YOUR_PUBLIC_IP>:/tmp/
   ```

   Note: We copy to `/tmp` first because `azureuser` doesn't have write access to `/opt`

4. **Back in your SSH session on the VM, move the JAR to the application directory:**

   ```bash
   sudo mv /tmp/hellojava-0.0.1-SNAPSHOT.jar /opt/hellojava/
   ```

5. **Create a symlink for easier management:**

   ```bash
   sudo ln -sf /opt/hellojava/hellojava-0.0.1-SNAPSHOT.jar /opt/hellojava/hellojava.jar
   ```

   This creates a generic `hellojava.jar` that points to the specific version

6. **Set proper ownership:**

   ```bash
   sudo chown -h hellojava:hellojava /opt/hellojava/hellojava-0.0.1-SNAPSHOT.jar /opt/hellojava/hellojava.jar
   ```

7. **Set appropriate permissions:**

   ```bash
   sudo chmod 500 /opt/hellojava/hellojava-0.0.1-SNAPSHOT.jar
   ```

   - `500` means: owner can read and execute, no one else can access

8. **Verify the setup:**

   ```bash
   ls -lh /opt/hellojava/
   ```

   Should show both the actual JAR and the symlink pointing to it

9. **Test the application runs:**

    ```bash
    sudo -u hellojava java -jar /opt/hellojava/hellojava.jar
    ```

    Wait for "Started HelloJavaApplication" then press `Ctrl+C` to stop it.

> ℹ **Concept Deep Dive**
>
> The two-step copy process (local → /tmp → /opt) is necessary because regular users can't write directly to `/opt`. The symlink approach (`hellojava.jar` → actual version) allows you to update to new versions without changing the systemd service configuration - just update the symlink and restart. File permissions `500` (r-x------) follow the principle of least privilege - the application only needs to read and execute the JAR, and other users don't need any access. Testing with `sudo -u hellojava` verifies the application works with the restricted user account before creating the service.
>
> ⚠ **Common Mistakes**
>
> - Leaving the JAR in `/tmp` - this directory is cleaned on reboot
> - Wrong permissions like 777 allow any user to modify or replace the application
> - Not testing before creating the service means systemd errors are harder to debug
> - Using regular `sudo java` runs as root, hiding permission problems
> - Forgetting `-h` flag in chown leaves symlink with wrong ownership
>
> ✓ **Quick check:** Application starts successfully when run as hellojava user, symlink points to correct JAR

### **Step 4:** Create systemd Service

Configure systemd to automatically start, stop, and restart your application as a managed service.

1. **Create a systemd service file:**

   ```bash
   sudo nano /etc/systemd/system/hellojava.service
   ```

2. **Add the following configuration:**

   ```ini
   [Unit]
   Description=HelloJava Spring Boot Application
   After=network.target

   [Service]
   Type=simple
   User=hellojava
   Group=hellojava
   WorkingDirectory=/opt/hellojava
   ExecStart=/usr/bin/java -jar /opt/hellojava/hellojava.jar
   SuccessExitStatus=143
   TimeoutStopSec=10
   Restart=on-failure
   RestartSec=5

   # Security settings
   NoNewPrivileges=true
   PrivateTmp=true

   [Install]
   WantedBy=multi-user.target
   ```

3. **Save and exit** (Ctrl+X, then Y, then Enter)

4. **Reload systemd to recognize the new service:**

   ```bash
   sudo systemctl daemon-reload
   ```

5. **Enable the service to start on boot:**

   ```bash
   sudo systemctl enable hellojava
   ```

6. **Start the service:**

   ```bash
   sudo systemctl start hellojava
   ```

7. **Check service status:**

   ```bash
   sudo systemctl status hellojava
   ```

   Should show: `Active: active (running)`

8. **View live application logs:**

   ```bash
   sudo journalctl -u hellojava -f
   ```

   Press `Ctrl+C` to stop viewing logs (service continues running)

> ℹ **Concept Deep Dive**
>
> systemd is the init system and service manager for modern Linux distributions. The `[Unit]` section defines dependencies (start after network is available). The `[Service]` section specifies how to run the application: as which user, from which directory, and what command to execute. `Type=simple` means the process started by `ExecStart` is the main service process. `Restart=on-failure` automatically restarts the application if it crashes. `SuccessExitStatus=143` tells systemd that exit code 143 (SIGTERM) is a clean shutdown, not a failure. The `[Install]` section controls how `systemctl enable` behaves - `WantedBy=multi-user.target` means start during normal system boot.
>
> **Security Settings Explained:**
> - `NoNewPrivileges=true` prevents the process from gaining new privileges through setuid/setgid
> - `PrivateTmp=true` gives the service its own private `/tmp` directory, isolated from other processes
>
> ⚠ **Common Mistakes**
>
> - Typos in the service file cause cryptic systemd errors - check with `systemctl status`
> - Wrong paths for `ExecStart` or `WorkingDirectory` prevent the service from starting
> - Forgetting `daemon-reload` after editing the service file means changes aren't picked up
> - Not enabling the service means it won't start automatically after reboot
> - Using relative paths in `ExecStart` doesn't work - always use absolute paths
>
> ✓ **Quick check:** `systemctl status hellojava` shows "active (running)" and no errors

### **Step 5:** Configure Firewall

Open port 8080 in the Azure Network Security Group so the application is accessible from the internet.

1. **In Azure Portal, navigate to your VM:** `hellojava-prod-vm`

2. **Go to Networking:** Left menu → "Networking" → "Network settings"

3. **Add inbound port rule:**
   - Click "Create port rule" → "Inbound port rule"
   - Destination port ranges: `8080`
   - Protocol: TCP
   - Action: Allow
   - Priority: 310
   - Name: `Allow-HTTP-8080`
   - Click "Add"

> ✓ **Quick check:** NSG rule appears with Status "Succeeded"

### **Step 6:** Test and Verify Production Deployment

Thoroughly test the production deployment including automatic restart capabilities and service management.

1. **Test web access from your browser:**

   ```
   http://<YOUR_PUBLIC_IP>:8080/
   ```

   Landing page should load

2. **Test the hello endpoint:**

   ```
   http://<YOUR_PUBLIC_IP>:8080/hello?name=Production
   ```

   Should display "Hello, Production!"

3. **Test service management commands:**

   **Check status:**
   ```bash
   sudo systemctl status hellojava
   ```

   **Restart service:**
   ```bash
   sudo systemctl restart hellojava
   ```

   Wait 5 seconds, then verify the app is accessible in browser

   **Stop service:**
   ```bash
   sudo systemctl stop hellojava
   ```

   Verify app is NOT accessible in browser (connection refused)

   **Start service:**
   ```bash
   sudo systemctl start hellojava
   ```

   Verify app is accessible again

4. **Test auto-restart on crash:**

   ```bash
   # Find the Java process ID
   sudo systemctl status hellojava
   ```

   Note the PID (process ID) shown after "Main PID:"

   ```bash
   # Kill the process to simulate a crash
   sudo kill -9 <PID>
   ```

   Wait 5 seconds, then check status:
   ```bash
   sudo systemctl status hellojava
   ```

   The service should have automatically restarted with a new PID!

5. **Test auto-start on system reboot:**

   ```bash
   sudo reboot
   ```

   Wait 2-3 minutes for the VM to reboot, then:

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   ```

   ```bash
   sudo systemctl status hellojava
   ```

   The service should be running automatically!

6. **View application logs:**

   ```bash
   # View last 50 lines
   sudo journalctl -u hellojava -n 50

   # Follow logs in real-time
   sudo journalctl -u hellojava -f

   # View logs since last boot
   sudo journalctl -u hellojava -b
   ```

> ✓ **Success indicators:**
>
> - Application is accessible via public IP
> - Service restarts successfully with `systemctl restart`
> - Service automatically restarts after crash (kill -9)
> - Service starts automatically after VM reboot
> - Logs are accessible via journalctl
> - No errors in `systemctl status` output
> - All endpoints work correctly (/hello, /hello?name=Test)

> ✓ **Final verification checklist:**
>
> - ☐ Application JAR deployed to /opt/hellojava with symlink
> - ☐ Application runs as hellojava system user
> - ☐ systemd service is enabled and active
> - ☐ Application survives crashes (auto-restart)
> - ☐ Application survives reboots (auto-start)
> - ☐ Logs are available via journalctl
> - ☐ All endpoints accessible from internet
> - ☐ Service management commands work (start, stop, restart, status)

## Common Issues

> **If you encounter problems:**
>
> **Service fails to start:** Check logs with `sudo journalctl -u hellojava -n 50`. Common causes: wrong Java path, permission issues, port already in use.
>
> **"Failed to start" error:** Verify the service file syntax with `sudo systemd-analyze verify hellojava.service`. Check for typos in paths.
>
> **Application starts but immediately stops:** Check for port conflicts (`sudo lsof -i :8080`) or Java heap size issues on B1s (only 1 GB RAM).
>
> **Service won't enable:** Check for errors in the `[Install]` section. Verify `multi-user.target` exists with `systemctl list-units --type=target`.
>
> **Logs don't appear in journalctl:** Ensure the service is actually running. Check that standard output isn't being redirected elsewhere.
>
> **Permission denied errors:** Verify `/opt/hellojava` ownership and permissions. Ensure hellojava user can read the JAR file.
>
> **Service doesn't start on reboot:** Check that it's enabled: `systemctl is-enabled hellojava` should return "enabled". Review boot logs: `journalctl -b`.
>
> **Still stuck?** Check full service status: `sudo systemctl status hellojava -l --no-pager` for complete output without truncation

## Production Considerations

Your deployment is now production-ready in terms of process management, but consider these additional improvements for a complete production system:

**Monitoring:** Set up application health checks and alerting (Azure Monitor, Application Insights)

**Logging:** Configure log rotation and centralized logging (Azure Log Analytics)

**Security:**
- Use Nginx as reverse proxy (port 80/443 instead of 8080)
- Add SSL/TLS certificates (Let's Encrypt)
- Implement rate limiting and request filtering
- Enable automatic security updates: `sudo apt install unattended-upgrades`

**Configuration:**
- Externalize configuration (application.properties from /etc or environment variables)
- Use Azure Key Vault for secrets
- Configure different profiles for different environments

**Backup:**
- Regular snapshots of the VM disk
- Backup of application state/data
- Disaster recovery plan

**Scaling:**
- Multiple VMs behind a load balancer
- Auto-scaling based on metrics
- Or migrate to Azure App Service or AKS for built-in scaling

## Summary

You've created a production-ready deployment with systemd:

- ✓ Dedicated system user for security
- ✓ systemd service for automatic management
- ✓ Auto-restart on crash
- ✓ Auto-start on reboot
- ✓ Centralized logging with journalctl
- ✓ Symlink-based deployment for easy updates

> **Key takeaway:** Production deployments require more than just "getting it running." Proper user accounts, service management, automatic restarts, and logging are essential for reliability. systemd provides all these capabilities on Linux and is the standard way to run services on modern distributions. The symlink approach makes it easy to deploy new versions without reconfiguring the service.

## Going Deeper (Optional)

> **Want to explore more?**
>
> - Configure JVM options for production (heap size, GC settings)
> - Set up log rotation with logrotate
> - Implement health checks and monitoring
> - Learn proper Maven release workflows (next tutorial!)
> - Configure Nginx reverse proxy for security
> - Set up multiple environments (staging, production)

## Done! 🎉

Outstanding work! You now have a production-ready Spring Boot deployment on Azure with automatic process management using systemd. Your application automatically restarts on crashes and starts on system boot - essential for production reliability.

**Next Steps:** In the next tutorial, you'll learn proper Maven release workflows to create immutable, versioned releases suitable for production deployments. After that, you'll enhance security by adding an Nginx reverse proxy.

## Clean Up

To avoid ongoing charges:

```bash
az group delete --name hellojava-prod-rg --yes --no-wait
```

This removes the VM, disk, network interface, NSG, and public IP - everything in the resource group.

## Appendix: Infrastructure as Code

For repeated deployments, automation scripts are available in the `scripts/04-configure-systemd-service/` directory.

> **Note:** The script examples shown below are for reference. Always use the actual script files in the `scripts/04-configure-systemd-service/` directory for the most up-to-date and tested versions.

**`scripts/04-configure-systemd-service/deploy-snapshot.sh`** - Automates SNAPSHOT deployment:

```bash
#!/bin/bash
# Automated SNAPSHOT deployment script
# Usage: bash scripts/04-configure-systemd-service/deploy-snapshot.sh <VM_PUBLIC_IP>

VM_IP="$1"
VM_USER="azureuser"
APP_USER="hellojava"
APP_DIR="/opt/hellojava"

if [ -z "$VM_IP" ]; then
    echo "Usage: $0 <VM_PUBLIC_IP>"
    exit 1
fi

echo "Building application..."
./mvnw clean package

# Get JAR filename
JAR_FILE=$(ls target/hellojava-*.jar 2>/dev/null | head -n 1)
if [ -z "$JAR_FILE" ]; then
    echo "Error: No JAR file found in target/"
    exit 1
fi

JAR_NAME=$(basename "$JAR_FILE")
echo "Deploying: $JAR_NAME"

# Copy to VM
scp "$JAR_FILE" "${VM_USER}@${VM_IP}:/tmp/"

# Deploy on VM
ssh "${VM_USER}@${VM_IP}" << EOF
    sudo systemctl stop hellojava || true
    sudo mv /tmp/${JAR_NAME} ${APP_DIR}/
    sudo chown ${APP_USER}:${APP_USER} ${APP_DIR}/${JAR_NAME}
    sudo chmod 500 ${APP_DIR}/${JAR_NAME}
    sudo ln -sf ${APP_DIR}/${JAR_NAME} ${APP_DIR}/hellojava.jar
    sudo systemctl start hellojava
    sudo systemctl status hellojava --no-pager
EOF

echo "Deployment complete!"
```

**`scripts/04-configure-systemd-service/setup-production-vm.sh`** - Complete production VM setup:

```bash
#!/bin/bash
# Complete production VM setup script for Tutorial 4
# Usage: bash scripts/04-configure-systemd-service/setup-production-vm.sh <VM_PUBLIC_IP>

VM_IP="$1"
VM_USER="azureuser"

if [ -z "$VM_IP" ]; then
    echo "Usage: $0 <VM_PUBLIC_IP>"
    exit 1
fi

echo "Configuring production environment on ${VM_IP}..."

ssh "${VM_USER}@${VM_IP}" << 'ENDSSH'
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-21-jre-headless
sudo useradd -r -s /bin/false hellojava || echo "User already exists"
sudo mkdir -p /opt/hellojava
sudo chown hellojava:hellojava /opt/hellojava

sudo tee /etc/systemd/system/hellojava.service > /dev/null << 'EOF'
[Unit]
Description=HelloJava Spring Boot Application
After=network.target

[Service]
Type=simple
User=hellojava
Group=hellojava
WorkingDirectory=/opt/hellojava
ExecStart=/usr/bin/java -jar /opt/hellojava/hellojava.jar
SuccessExitStatus=143
TimeoutStopSec=10
Restart=on-failure
RestartSec=5

NoNewPrivileges=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable hellojava
echo "Production environment configured!"
ENDSSH

echo "Done! Deploy with: bash scripts/04-configure-systemd-service/deploy-snapshot.sh ${VM_IP}"
```

**`scripts/04-configure-systemd-service/provision-vm.sh`** - Provision Azure VM with cloud-init:

```bash
#!/bin/bash
# Provision Azure VM for production environment
# Usage: cd scripts/04-configure-systemd-service && bash provision-vm.sh

RESOURCE_GROUP="hellojava-prod-rg"
VM_NAME="hellojava-prod-vm"
LOCATION="northeurope"
IMAGE="Ubuntu2404"
SIZE="Standard_B1s"
CLOUD_INIT_FILE="cloud-init.yaml"

# Check that you are in the same directory as the cloud-init file
if [ ! -f $CLOUD_INIT_FILE ]; then
  echo "Error: $CLOUD_INIT_FILE not found!"
  echo "Make sure you run this script from the scripts/04-configure-systemd-service directory"
  exit 1
fi

echo "Creating resource group..."
az group create --name $RESOURCE_GROUP --location $LOCATION

echo "Creating VM with cloud-init..."
PUBLIC_IP=$(az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image $IMAGE \
  --size $SIZE \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard \
  --custom-data @$CLOUD_INIT_FILE \
  --query publicIpAddress \
  --output tsv)

echo "Opening port 8080..."
az vm open-port --resource-group $RESOURCE_GROUP --name $VM_NAME --port 8080 --priority 310

echo ""
echo "VM created successfully!"
echo "Public IP: $PUBLIC_IP"
echo ""
echo "Next steps:"
echo "1. Wait 1-2 minutes for cloud-init to complete"
echo "2. Run: bash scripts/04-configure-systemd-service/setup-production-vm.sh $PUBLIC_IP"
```

### Alternative: Cloud-Init Approach

Instead of using SSH to configure the VM after creation, you can automate everything using cloud-init. This demonstrates Infrastructure as Code principles.

**`scripts/04-configure-systemd-service/cloud-init.yaml`** - Complete VM setup via cloud-init:

```yaml
#cloud-config
# Complete production VM setup using cloud-init
# This automates all steps from setup-production-vm.sh

# Update packages and install Java
package_update: true
package_upgrade: true
packages:
  - openjdk-21-jre-headless

# Create hellojava system user
users:
  - name: hellojava
    system: true
    shell: /bin/false
    no_create_home: true

# Create directory structure
runcmd:
  # Create application directory
  - mkdir -p /opt/hellojava
  - chown hellojava:hellojava /opt/hellojava
  - chmod 755 /opt/hellojava

  # Verify Java installation
  - java -version

  # Enable the systemd service (will be started when JAR is deployed)
  - systemctl daemon-reload
  - systemctl enable hellojava

  # Log completion
  - echo "Production environment setup complete" | tee -a /var/log/cloud-init-output.log

# Create systemd service file
write_files:
  - path: /etc/systemd/system/hellojava.service
    owner: root:root
    permissions: '0644'
    content: |
      [Unit]
      Description=HelloJava Spring Boot Application
      After=network.target

      [Service]
      Type=simple
      User=hellojava
      Group=hellojava
      WorkingDirectory=/opt/hellojava
      ExecStart=/usr/bin/java -jar /opt/hellojava/hellojava.jar
      SuccessExitStatus=143
      TimeoutStopSec=10
      Restart=on-failure
      RestartSec=5

      # Security settings
      NoNewPrivileges=true
      PrivateTmp=true

      [Install]
      WantedBy=multi-user.target

# Optional: Set timezone
timezone: Europe/Oslo

# Optional: Final message
final_message: "HelloJava production environment ready after $UPTIME seconds"
```

**Using cloud-init workflow:**

```bash
# 1. Provision VM with cloud-init (does Steps 2-5 automatically)
cd scripts/04-configure-systemd-service && bash provision-vm.sh
# Note the public IP
cd ../..

# 2. Wait for cloud-init to complete (1-2 minutes)
sleep 90

# 3. Deploy application (skip setup-production-vm.sh - already done by cloud-init!)
bash scripts/04-configure-systemd-service/deploy-snapshot.sh <PUBLIC_IP>
```

**Advantages of cloud-init approach:**

- **Declarative:** Describe desired state, not steps
- **Idempotent:** Safe to run multiple times
- **Faster:** Runs during VM boot, no SSH needed
- **Version controlled:** Infrastructure configuration in YAML
- **Cloud-native:** Works across Azure, AWS, GCP

**Verify cloud-init completion:**

```bash
# SSH into VM
ssh azureuser@<PUBLIC_IP>

# Check cloud-init status
cloud-init status

# View cloud-init logs
sudo cat /var/log/cloud-init-output.log
```

**Complete first-time deployment workflow (traditional SSH approach):**

```bash
# 1. Provision VM
cd scripts/04-configure-systemd-service && bash provision-vm.sh
# Note the public IP
cd ../..

# 2. Wait for cloud-init (1-2 minutes)
sleep 90

# 3. Configure for production
bash scripts/04-configure-systemd-service/setup-production-vm.sh <PUBLIC_IP>

# 4. Deploy application
bash scripts/04-configure-systemd-service/deploy-snapshot.sh <PUBLIC_IP>
```

> **When to use automation:**
>
> - **Development teams:** Consistent deployments across team members
> - **Frequent updates:** Deploy changes quickly during development
> - **Multiple environments:** Deploy to different VMs with one command
> - **Testing:** Rapid iteration and testing cycles
>
> **When to deploy manually:**
>
> - **Learning phase:** Understanding each step before automating
> - **One-time deployments:** Overhead of creating scripts not worth it
> - **Troubleshooting:** Manual control helps identify issues


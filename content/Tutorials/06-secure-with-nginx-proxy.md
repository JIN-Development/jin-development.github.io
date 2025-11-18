+++
title = "6. Secure Your Application with Nginx Reverse Proxy"
weight = 6
date = 2025-11-11
draft = false
tags = ["week1", "JIN"]
+++

## Goal

Add an Nginx reverse proxy in front of your Spring Boot application to improve security, prepare for SSL/TLS, and follow production best practices. Hide your application server behind a professional web server that handles HTTP traffic.

> **What you'll learn:**
>
> - How to configure Nginx as a reverse proxy for Java applications
> - Why reverse proxies improve security and performance
> - How to lock down network access to application ports
> - Best practices for production web application architecture
> - How to prepare infrastructure for SSL/TLS certificates

## Prerequisites

> **Before starting, ensure you have:**
>
> - ✓ Completed "Maven Release Process and Git Workflow" tutorial
> - ✓ Spring Boot application running with systemd on Azure VM
> - ✓ Application accessible on port 8080
> - ✓ SSH access to your Azure VM

## Exercise Steps

### Overview

1. **Understand Reverse Proxy Architecture**
2. **Install and Configure Nginx**
3. **Configure Application to Bind Localhost Only**
4. **Update Azure Network Security Group**
5. **Test and Verify Secure Configuration**

### **Step 1:** Understand Reverse Proxy Architecture

Before implementing, understand what a reverse proxy does and why it's essential for production deployments.

> ℹ **Concept Deep Dive**
>
> **What is a reverse proxy?**
>
> A reverse proxy sits between clients (browsers) and your application server. Instead of clients connecting directly to your Java application on port 8080, they connect to Nginx on port 80, and Nginx forwards the requests to your application.
>
> ```
> Before (Direct Access):
> Browser → Internet → Azure NSG :8080 → Java App :8080
>
> After (Reverse Proxy):
> Browser → Internet → Azure NSG :80 → Nginx :80 → Java App :127.0.0.1:8080
> ```
>
> **Why use a reverse proxy?**
>
> 1. **Security:** Application server hidden from direct internet access
> 2. **Standard ports:** Use port 80 (HTTP) and 443 (HTTPS) instead of 8080
> 3. **SSL termination:** Nginx handles SSL/TLS certificates (future tutorial)
> 4. **Performance:** Nginx can cache static content, compress responses
> 5. **Load balancing:** Distribute traffic across multiple application instances
> 6. **Security headers:** Add HTTP security headers easily
> 7. **DDoS protection:** Rate limiting and connection throttling
> 8. **Separation of concerns:** Web server handles HTTP, app server handles business logic
>
> **Industry standard:** Major websites use reverse proxies (Nginx, Apache, HAProxy) in front of application servers. This is the foundation of modern web architecture.
>
> ✓ **Quick check:** You understand why reverse proxies are used in production

### **Step 2:** Install and Configure Nginx

Install Nginx on your Azure VM and configure it to proxy requests to your Spring Boot application.

1. **SSH into your Azure VM:**

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   ```

2. **Update package lists:**

   ```bash
   sudo apt update
   ```

3. **Install Nginx:**

   ```bash
   sudo apt install -y nginx
   ```

4. **Verify Nginx is running:**

   ```bash
   sudo systemctl status nginx
   ```

   Should show: `Active: active (running)`

5. **Test Nginx default page:**

   Open your browser and navigate to:

   ```
   http://<YOUR_PUBLIC_IP>
   ```

   You should see the "Welcome to nginx!" default page

6. **Create Nginx configuration** for your application:

   ```bash
   sudo nano /etc/nginx/sites-available/hellojava
   ```

7. **Add the following configuration:**

   ```nginx
   server {
       listen 80;
       listen [::]:80;

       server_name _;

       # Proxy to Java application
       location / {
           proxy_pass http://127.0.0.1:8080;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;

           # Timeouts
           proxy_connect_timeout 60s;
           proxy_send_timeout 60s;
           proxy_read_timeout 60s;
       }

       # Health check endpoint (optional)
       location /health {
           access_log off;
           proxy_pass http://127.0.0.1:8080/actuator/health;
       }
   }
   ```

8. **Save and exit** (Ctrl+X, then Y, then Enter)

9. **Enable the site** by creating a symbolic link:

   ```bash
   sudo ln -s /etc/nginx/sites-available/hellojava /etc/nginx/sites-enabled/
   ```

10. **Remove the default site** (optional but recommended):

    ```bash
    sudo rm /etc/nginx/sites-enabled/default
    ```

11. **Test Nginx configuration** for syntax errors:

    ```bash
    sudo nginx -t
    ```

    Should show: `syntax is ok` and `test is successful`

12. **Reload Nginx** to apply configuration:

    ```bash
    sudo systemctl reload nginx
    ```

> ℹ **Concept Deep Dive**
>
> **Nginx configuration breakdown:**
>
> - `listen 80` - Listen on port 80 (HTTP)
> - `server_name _` - Accept requests for any domain (catch-all)
> - `proxy_pass` - Forward requests to this upstream server (your Java app)
> - `proxy_set_header Host` - Preserve the original Host header
> - `proxy_set_header X-Real-IP` - Pass client's real IP to application
> - `proxy_set_header X-Forwarded-For` - Standard header for proxy chains
> - `proxy_set_header X-Forwarded-Proto` - Tell app if request was HTTP/HTTPS
>
> **Why these headers matter:**
>
> Without these headers, your Java application would see all requests coming from 127.0.0.1 (Nginx) instead of the actual client IP. These headers preserve the original request information.
>
> **Sites-available vs sites-enabled:**
>
> - `sites-available/` - Configuration files (storage)
> - `sites-enabled/` - Symbolic links to active configurations
> - This pattern allows you to disable sites without deleting configuration
>
> **Testing configuration before reloading:**
>
> Always run `nginx -t` before reloading. A configuration error can break your web server. The test command validates syntax without affecting the running server.
>
> ⚠ **Common Mistakes**
>
> - Forgetting semicolons at end of directives causes syntax errors
> - Typo in `proxy_pass` URL (must be http://127.0.0.1:8080, not localhost)
> - Not creating symlink means configuration won't be loaded
> - Reloading without testing can break running server
> - Wrong file permissions on config file (should be readable by nginx user)
>
> ✓ **Quick check:** `nginx -t` shows success, default nginx page replaced with your app

### **Step 3:** Configure Application to Bind Localhost Only

Enhance security by configuring your Spring Boot application to only accept connections from localhost (127.0.0.1), preventing direct external access. We'll use environment variables for configuration following the 12-Factor App methodology.

1. **Create environment configuration** directory:

   ```bash
   sudo mkdir -p /etc/hellojava
   ```

2. **Create environment file:**

   ```bash
   sudo nano /etc/hellojava/environment
   ```

3. **Add the following configuration:**

   ```bash
   # HelloJava Production Environment Configuration
   # This file is loaded by systemd and sets environment variables for the application

   # Server Configuration - Bind to localhost only (behind Nginx)
   SERVER_ADDRESS=127.0.0.1
   SERVER_PORT=8080

   # Trust proxy headers from Nginx
   SERVER_FORWARD_HEADERS_STRATEGY=native

   # Logging
   LOGGING_LEVEL_ROOT=INFO
   ```

4. **Save and exit** (Ctrl+X, then Y, then Enter)

5. **Set proper permissions:**

   ```bash
   sudo chmod 600 /etc/hellojava/environment
   sudo chown root:root /etc/hellojava/environment
   ```

6. **Update systemd service** to load the environment file:

   ```bash
   sudo nano /etc/systemd/system/hellojava.service
   ```

7. **Add the EnvironmentFile line** after `WorkingDirectory`:

   Find:

   ```ini
   WorkingDirectory=/opt/hellojava
   ExecStart=/usr/bin/java -jar /opt/hellojava/hellojava.jar
   ```

   Change to:

   ```ini
   WorkingDirectory=/opt/hellojava
   EnvironmentFile=/etc/hellojava/environment
   ExecStart=/usr/bin/java -jar /opt/hellojava/hellojava.jar
   ```

8. **Save and exit**

9. **Reload systemd** to recognize the service change:

   ```bash
   sudo systemctl daemon-reload
   ```

10. **Restart the application:**

    ```bash
    sudo systemctl restart hellojava
    ```

11. **Verify the service started successfully:**

    ```bash
    sudo systemctl status hellojava
    ```

12. **Test that application is now on localhost only:**

    From the VM, test localhost access (should work):

    ```bash
    curl http://127.0.0.1:8080/hello
    ```

    Try to access from public IP on port 8080 (should fail):

    ```bash
    curl http://<YOUR_PUBLIC_IP>:8080/hello
    ```

    This should timeout or be refused (expected!)

> ℹ **Concept Deep Dive**
>
> **Environment Variables for Configuration (12-Factor App):**
>
> We use environment variables instead of properties files following the 12-Factor App methodology. Spring Boot automatically maps environment variables to properties:
> - `SERVER_ADDRESS` → `server.address`
> - `SERVER_PORT` → `server.port`
> - `SERVER_FORWARD_HEADERS_STRATEGY` → `server.forward-headers-strategy`
>
> The naming convention uses underscores and uppercase (environment standard) instead of dots and lowercase (Java properties standard).
>
> **Why `/etc/hellojava/environment`?**
>
> Placing configuration in `/etc` follows Linux conventions - this is where system-wide configuration lives (like `/etc/nginx/`, `/etc/postgresql/`). The environment file pattern is used by many production services and makes configuration visible and maintainable. It's similar to a `.env` file in development but with proper Linux permissions (600 = only root can read).
>
> **Binding to localhost (127.0.0.1):**
>
> By default, Spring Boot binds to `0.0.0.0`, which means "all network interfaces" - the application accepts connections from any IP address. Setting `SERVER_ADDRESS=127.0.0.1` restricts it to only accept connections from the same machine.
>
> ```
> 0.0.0.0:8080  → Accepts from anywhere (public internet, localhost, etc.)
> 127.0.0.1:8080 → Accepts from localhost only (Nginx, not internet)
> ```
>
> This is a critical security hardening step. Even if someone finds an NSG misconfiguration or firewall hole, they can't directly access your application - they must go through Nginx.
>
> **EnvironmentFile directive:**
>
> `EnvironmentFile=/etc/hellojava/environment` tells systemd to load environment variables from this file before starting the service. The environment file can be updated and the service restarted without running `systemctl daemon-reload` - only the service definition file (`.service`) requires daemon-reload when changed.
>
> **Forward headers strategy:**
>
> `SERVER_FORWARD_HEADERS_STRATEGY=native` tells Spring Boot to trust proxy headers (`X-Forwarded-*`). This is safe because only Nginx (running on localhost) can reach our application. Without this, Spring Boot wouldn't recognize the original client IP or protocol.
>
> **Why Environment Variables Instead of Properties Files?**
>
> There are three common configuration approaches:
>
> | Approach | Storage | Security | Portability | When to Use |
> |----------|---------|----------|-------------|------------|
> | **Properties Files in JAR** | Compiled into `.jar` | Poor - hardcoded secrets | Poor - recompile per environment | Development only |
> | **External Properties Files** | `/opt/hellojava/application.properties` | Medium - file permissions limited | Medium - paths are machine-specific | Legacy deployments |
> | **Environment Variables** | systemd environment file | Good - Linux enforces 600 permissions | Excellent - works everywhere | Production standard |
>
> **Example comparison:**
>
> Properties file approach (risky):
> ```
> server.address=127.0.0.1
> server.port=8080
> database.password=SecretPassword123  # SECURITY RISK - in source control!
> ```
>
> Environment variable approach (secure):
> ```
> SERVER_ADDRESS=127.0.0.1
> SERVER_PORT=8080
> DATABASE_PASSWORD=SecretPassword123  # Secured with Linux permissions (600)
> ```
>
> **Key advantages of environment variables:**
> - **Platform agnostic:** Work identically on VMs, Docker, Kubernetes, and cloud platforms
> - **Security:** Linux file permissions (600) ensure only root can read secrets
> - **CI/CD friendly:** Standard practice for deployment pipelines and secret management
> - **Secrets integration:** Easy to integrate with Azure Key Vault, Kubernetes Secrets, or HashiCorp Vault
> - **No recompilation:** Update configuration and restart service without rebuilding JAR
> - **12-Factor App compliant:** Strictly separates code (immutable JAR) from configuration (environment-specific)
>
> **Traditional Alternative:**
>
> An older approach uses `--spring.config.additional-location=file:/opt/hellojava/config/application.properties`. This works but requires file path management and isn't container-friendly. Environment variables are preferred because they work identically across VMs, containers, Kubernetes, and cloud platforms.
>
> ⚠ **Common Mistakes**
>
> - Typo in environment variable names (must be uppercase with underscores)
> - Wrong bind address (127.0.0.1, not 127.0.0.0 or localhost)
> - Forgetting to reload systemd after editing service file
> - Not testing localhost access before locking down NSG
> - Making environment file world-readable (should be 600, not 644)
> - Enabling forward-headers-strategy on internet-facing apps (security risk)
>
> ✓ **Quick check:** Application not accessible via public IP:8080, but works via Nginx on port 80

### **Step 4:** Update Azure Network Security Group

Lock down the Azure Network Security Group (NSG) to only allow traffic on ports 22 (SSH) and 80 (HTTP), removing direct access to port 8080.

1. **In Azure Portal, navigate to your VM:** `hellojava-prod-vm`

2. **Go to Networking:** Left menu → "Networking" → "Network settings"

3. **Identify the port 8080 rule:** Look for the rule named `Allow-HTTP-8080` or similar

4. **Delete the port 8080 rule:**
   - Click on the rule
   - Click "Delete"
   - Confirm deletion

5. **Verify port 80 is allowed:**

   Check if there's already a rule for port 80. If not, create one:

   - Click "Create port rule" → "Inbound port rule"
   - Destination port ranges: `80`
   - Protocol: TCP
   - Action: Allow
   - Priority: 300
   - Name: `Allow-HTTP-80`
   - Click "Add"

6. **Verify final NSG rules:**

   You should have:
   - ☐ SSH (22) - Allow
   - ☐ HTTP (80) - Allow
   - ☐ HTTP (8080) - Deleted/None

> ℹ **Concept Deep Dive**
>
> **Azure Network Security Group (NSG):**
>
> NSGs are Azure's network firewall, controlling inbound and outbound traffic to Azure resources. They act as the first line of defense, filtering traffic before it reaches your VM.
>
> **Defense in depth strategy:**
>
> We're implementing multiple security layers:
> 1. **NSG** - Azure network firewall (blocks 8080 at network level)
> 2. **Application binding** - Java app only listens on 127.0.0.1 (OS level)
> 3. **Nginx** - Reverse proxy controlling access patterns (application level)
>
> Even if one layer fails, others provide protection. This is called "defense in depth" and is a fundamental security principle.
>
> **Priority numbers:**
>
> NSG rules are evaluated by priority (lower numbers first). We use 300 for HTTP and 310 for SSH, leaving room to add rules in between (like 305) if needed later.
>
> **Why delete port 8080 entirely?**
>
> Leaving port 8080 open "just in case" defeats the purpose. If you need direct access for debugging, temporarily add the rule, then remove it when done. Production systems should minimize exposed ports.
>
> ⚠ **Common Mistakes**
>
> - Deleting SSH rule (port 22) locks you out of the VM
> - Forgetting to add port 80 rule before deleting 8080
> - Wrong protocol (TCP vs UDP) - use TCP for HTTP
> - Too permissive source (0.0.0.0/0) - consider restricting to your IP for SSH
>
> ✓ **Quick check:** Port 80 allowed, port 8080 removed, SSH still accessible

### **Step 5:** Test and Verify Secure Configuration

Thoroughly test the new reverse proxy setup and verify security hardening is working correctly.

1. **Test application via HTTP (port 80):**

   Open your browser and navigate to:

   ```
   http://<YOUR_PUBLIC_IP>/
   ```

   Landing page should load (via Nginx, not direct)

2. **Test hello endpoint:**

   ```
   http://<YOUR_PUBLIC_IP>/hello?name=Nginx
   ```

   Should display: "Hello, Nginx!"

3. **Verify port 8080 is blocked:**

   Try to access the application directly:

   ```
   http://<YOUR_PUBLIC_IP>:8080/
   ```

   Should timeout or connection refused (expected! This is what we want)

4. **Test from command line:**

   ```bash
   # Should work (port 80 via Nginx)
   curl -I http://<YOUR_PUBLIC_IP>/

   # Should fail (port 8080 blocked)
   curl --max-time 5 http://<YOUR_PUBLIC_IP>:8080/
   ```

5. **Check Nginx logs:**

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   sudo tail -f /var/log/nginx/access.log
   ```

   Make a few requests in your browser, you should see them appear in the log

   Press `Ctrl+C` to stop

6. **Verify response headers:**

   ```bash
   curl -I http://<YOUR_PUBLIC_IP>/
   ```

   Look for `Server: nginx` header (proves requests go through Nginx)

7. **Test service restart:**

   ```bash
   sudo systemctl restart hellojava
   sleep 5
   curl http://<YOUR_PUBLIC_IP>/hello
   ```

   Should work after restart

8. **Test Nginx reload:**

   ```bash
   sudo systemctl reload nginx
   curl http://<YOUR_PUBLIC_IP>/hello
   ```

   Should work without interruption (Nginx reload is graceful)

> ✓ **Success indicators:**
>
> - Application accessible via HTTP on port 80
> - Port 8080 is blocked from internet access
> - All endpoints work through Nginx (/hello, /hello?name=Test)
> - Nginx logs show incoming requests
> - Response headers include `Server: nginx`
> - Application survives restart
> - SSH access still works on port 22

> ✓ **Final verification checklist:**
>
> - ☐ Nginx installed and running
> - ☐ Nginx configured as reverse proxy to localhost:8080
> - ☐ Application bound to 127.0.0.1 only
> - ☐ NSG allows port 80 (HTTP)
> - ☐ NSG blocks port 8080 (removed rule)
> - ☐ Application accessible via public IP on port 80
> - ☐ Application NOT accessible via public IP on port 8080
> - ☐ All application endpoints work correctly
> - ☐ Nginx logs show traffic

## Test Your Secure Configuration

Comprehensive security testing to verify your reverse proxy setup is working correctly.

1. **Security scan from external machine:**

   From your local computer (not the VM):

   ```bash
   # Test port 80 (should succeed)
   nmap -p 80 <YOUR_PUBLIC_IP>

   # Test port 8080 (should be filtered/closed)
   nmap -p 8080 <YOUR_PUBLIC_IP>
   ```

2. **Verify localhost binding:**

   SSH into the VM and check what ports are listening:

   ```bash
   sudo netstat -tlnp | grep java
   ```

   Should show: `127.0.0.1:8080` (not `0.0.0.0:8080`)

3. **Test proxy headers:**

   Check that your application receives correct headers:

   ```bash
   # From your local machine
   curl -H "X-Custom-Header: test" http://<YOUR_PUBLIC_IP>/hello
   ```

   Check Java application logs to see if headers are received

4. **Load test (optional):**

   ```bash
   # Simple load test with Apache Bench (if installed)
   ab -n 1000 -c 10 http://<YOUR_PUBLIC_IP>/hello
   ```

> ✓ **Success criteria:**
>
> - Port 80 shows as "open" in nmap
> - Port 8080 shows as "filtered" or "closed" in nmap
> - Java listens on 127.0.0.1:8080 (not 0.0.0.0)
> - Nginx handles requests without errors
> - Application logs show correct client headers

## Common Issues

> **If you encounter problems:**
>
> **502 Bad Gateway error:** Nginx can't reach the Java application. Check:
> - Is the Java app running? `sudo systemctl status hellojava`
> - Is it listening on 127.0.0.1:8080? `sudo netstat -tlnp | grep 8080`
> - Can you curl localhost from the VM? `curl http://127.0.0.1:8080/hello`
>
> **Application still accessible on port 8080:** The NSG rule wasn't properly deleted, or there's a caching issue. Check NSG rules in Azure Portal and wait 60 seconds for changes to propagate.
>
> **Nginx "address already in use" error:** Something else is using port 80. Check with:
> ```bash
> sudo lsof -i :80
> ```
> If it's the old default Nginx, restart: `sudo systemctl restart nginx`
>
> **"Connection refused" on port 80:** Nginx isn't running. Check:
> ```bash
> sudo systemctl status nginx
> sudo nginx -t  # Check configuration syntax
> ```
>
> **Application can't start after config change:** Check the systemd service file path is correct. View logs:
> ```bash
> sudo journalctl -u hellojava -n 50
> ```
>
> **SSH connection lost:** If you accidentally blocked port 22, you'll need to use Azure Portal's "Serial Console" or "Reset password" feature to regain access.
>
> **Nginx configuration test fails:** Check for syntax errors:
> ```bash
> sudo nginx -t
> ```
> Common issues: missing semicolon, typo in proxy_pass URL, invalid directive.

## Summary

You've successfully secured your application with a reverse proxy:

- ✓ Nginx installed and configured as reverse proxy
- ✓ Application bound to localhost only (127.0.0.1)
- ✓ Azure NSG hardened (only ports 22 and 80 exposed)
- ✓ Defense in depth security layers implemented
- ✓ Production-ready architecture foundation
- ✓ Prepared for SSL/TLS certificates (future tutorial)

> **Key takeaway:** Reverse proxies are a fundamental component of production web architecture. By placing Nginx in front of your application, you've added security (hiding the app server), improved architecture (separation of concerns), and prepared for future enhancements like SSL/TLS, load balancing, and caching. This pattern is used by virtually all production web applications, from startups to enterprises.

## Going Deeper (Optional)

> **Want to explore more?**
>
> - Add SSL/TLS certificates with Let's Encrypt (certbot)
> - Configure Nginx caching for static resources
> - Implement rate limiting to prevent abuse
> - Add security headers (HSTS, CSP, X-Frame-Options)
> - Set up Nginx logging to Azure Log Analytics
> - Configure Nginx as a load balancer for multiple app instances
> - Implement basic authentication for admin endpoints
> - Use Nginx gzip compression to reduce bandwidth

## Done! 🎉

Outstanding work! You've implemented a production-grade reverse proxy setup that significantly improves your application's security and architecture. Your application is now hidden behind Nginx, only essential ports are exposed, and you're ready for SSL/TLS encryption.

**Next Steps:** Future tutorials will cover SSL/TLS certificates, containerization with Docker, and cloud-native patterns. You've built a solid foundation that scales from small projects to enterprise applications.

## Clean Up

No additional cleanup needed beyond the existing VM resources.

To remove everything:

```bash
az group delete --name hellojava-prod-rg --yes --no-wait
```

## Appendix: Automation Script

For automated Nginx setup on new VMs, see `scripts/06-secure-with-nginx-proxy/` directory.

This script can be used to:

- Install and configure Nginx automatically
- Update NSG rules via Azure CLI
- Set up application configuration
- Restart services

See Tutorial 6 automation scripts for complete infrastructure-as-code approach.

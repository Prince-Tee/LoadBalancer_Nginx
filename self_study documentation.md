#### **Project Overview**
In this project, I configured an **Nginx Load Balancer** on AWS, secured the traffic with **SSL/TLS** certificates using Let’s Encrypt, and automated certificate renewal. The solution was implemented as part of the **DevOps Cloud Engineering program by StegHub**. The goal was to set up a reliable and secure load balancing solution using **Nginx**, ensuring high availability and secure communication over **HTTPS**.


### **1. Uninstall Apache**

Since the initial server was using Apache, I started by uninstalling Apache to replace it with Nginx. Here’s how I did it:

**Steps:**
1. SSH into the EC2 instance:
   ```bash
   ssh ubuntu@your-server-ip
   ```
2. Stop the Apache service:
   ```bash
   sudo systemctl stop apache2
   ```
3. Uninstall Apache and its utilities:
   ```bash
   sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
   ```
4. Remove unused dependencies:
   ```bash
   sudo apt-get autoremove
   ```
5. Ensure no residual Apache files remain:
   ```bash
   sudo rm -rf /etc/apache2 /var/www/html
   ```

After removing Apache, I verified that it was completely uninstalled by checking the Apache version:
```bash
apache2 -v
```
If Apache was properly removed, an error or "command not found" was returned.

---

### **2. Install and Configure Nginx**

Next, I installed and configured **Nginx** as the Load Balancer to distribute incoming traffic across multiple web servers.

**Steps:**
1. Update and install Nginx:
   ```bash
   sudo apt update
   sudo apt install nginx
   ```

2. Open the default Nginx configuration file:
   ```bash
   sudo vi /etc/nginx/nginx.conf
   ```

3. Insert the following configuration into the `http` section to define upstream servers and load balancing logic:
   ```nginx
   upstream myproject {
      server Web1 weight=5;
      server Web2 weight=5;
   }

   server {
      listen 80;
      server_name www.domain.com;
      location / {
         proxy_pass http://myproject;
      }
   }
   ```

4. Comment out this line in the configuration file to avoid site conflicts:
   ```nginx
   # include /etc/nginx/sites-enabled/*;
   ```

5. Restart Nginx to apply the changes:
   ```bash
   sudo systemctl restart nginx
   sudo systemctl status nginx
   ```

---

### **3. Assign an Elastic IP to EC2**

To ensure that the public IP address of the EC2 instance remains static after a reboot, I assigned an **Elastic IP** to the Nginx Load Balancer.

**Steps:**
1. In the AWS Management Console, navigate to **Elastic IPs** under the EC2 dashboard.
2. Allocate a new Elastic IP address.
3. Associate the Elastic IP with the EC2 instance hosting Nginx.

---

### **4. Set Up FreeDNS Subdomain**

To point a domain name to my Nginx Load Balancer, I used a **FreeDNS** subdomain.

**Steps:**
1. Register and log in to [FreeDNS](https://freedns.afraid.org).
2. Select a free subdomain (e.g., `yourname.mooo.com`).
3. Go to the **DNS** section of FreeDNS and create an A Record pointing the subdomain to the EC2 Elastic IP:
   - **Type**: A
   - **Name**: yourname.mooo.com
   - **Destination**: The Elastic IP

I could now access the Nginx Load Balancer through the domain: `http://yourname.mooo.com`.

---

### **5. Configure SSL/TLS with Let’s Encrypt**

To secure traffic, I used **Let’s Encrypt** to generate free SSL/TLS certificates for the Nginx Load Balancer.

**Steps:**
1. Make sure that the **snapd** service is running:
   ```bash
   sudo systemctl status snapd
   ```

2. Install Certbot:
   ```bash
   sudo snap install --classic certbot
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```

3. Request an SSL/TLS certificate for the domain:
   ```bash
   sudo certbot --nginx
   ```

Certbot automatically updates the Nginx configuration to serve the domain over HTTPS.

---

### **6. Automate SSL/TLS Certificate Renewal**

Let’s Encrypt certificates expire after 90 days, so I set up an automated process to renew them.

**Steps:**
1. Open the crontab file:
   ```bash
   crontab -e
   ```

2. Add the following line to schedule the Certbot renewal to run twice daily:
   ```bash
   * */12 * * *   /usr/bin/certbot renew > /dev/null 2>&1
   ```

3. Test the renewal process with a dry run:
   ```bash
   sudo certbot renew --dry-run
   ```

---

### **7. Conclusion**

By following these steps, I successfully configured an Nginx Load Balancer with SSL/TLS encryption. The solution ensures high availability, secure communication, and automated certificate renewal, making it a robust setup for production environments.

---

### **8. Challenges Faced**

- **Apache Uninstallation**: Ensuring all Apache files were removed and no conflicting services were running before Nginx installation.
- **Elastic IP Configuration**: Understanding how to associate Elastic IP to prevent IP changes after instance restarts.
- **SSL/TLS Certificate Setup**: Navigating through Certbot configuration and automating renewal with crontab.
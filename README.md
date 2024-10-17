# Nginx Load Balancer Solution with SSL/TLS

## Table of Contents
1. [Uninstall Apache](#uninstall-apache)
2. [Install and Configure Nginx as Load Balancer](#install-nginx)
3. [Register and Use FreeDNS Subdomain](#register-freedns)
4. [Assign Elastic IP and Point FreeDNS Subdomain](#assign-elastic-ip)
5. [Configure Nginx to Recognize the Domain](#configure-nginx)
6. [Install SSL Certificate with Certbot](#install-certbot)
7. [Set Up Automatic Renewal of SSL Certificates](#automatic-renewal)
8. [Conclusion](#conclusion)

---

## <a name="uninstall-apache"></a> 1. Uninstall Apache

Lets uninstall Apache from the existing Load Balancer server setting up Nginx:

1. **SSH into your server:**

    ```bash
    ssh username@your-server-ip
    ```

2. **Stop the Apache service:**

    ```bash
    sudo systemctl stop apache2
    ```

3. **Uninstall Apache:**

    ```bash
    sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
    ```

4. **Remove unused dependencies:**

    ```bash
    sudo apt-get autoremove
    ```

5. **Check and remove any remaining Apache files:**

    ```bash
    sudo rm -rf /etc/apache2 /var/www/html
    ```

6. **Verify Apache removal:**

    ```bash
    apache2 -v
    ```

## <a name="install-nginx"></a> 2. Install and Configure Nginx as Load Balancer

1. **Update your system and install Nginx:**

    ```bash
    sudo apt update
    sudo apt install nginx
    ```

    _Screenshot placeholder for Nginx installation step_

2. **Open the Nginx configuration file:**

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

3. **Update the Nginx configuration:**

    Add the following inside the `http` section to set up load balancing between Web1 and Web2:

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

    _Screenshot placeholder for the Nginx config update_

4. **Comment out the following line:**

    ```bash
    #       include /etc/nginx/sites-enabled/*;
    ```

5. **Restart Nginx:**

    ```bash
    sudo systemctl restart nginx
    sudo systemctl status nginx
    ```

    _Screenshot placeholder for Nginx status check_

## <a name="register-freedns"></a> 3. Register and Use FreeDNS Subdomain

1. **Register a subdomain:**
   - Go to [FreeDNS](https://freedns.afraid.org) and register.
   - Choose a subdomain (e.g., `yourname.mooo.com`).

    _Screenshot placeholder for FreeDNS registration_

## <a name="assign-elastic-ip"></a> 4. Assign Elastic IP and Point FreeDNS Subdomain

1. **Allocate Elastic IP:**
   - In the AWS EC2 dashboard, click on **Elastic IPs**.
   - Allocate an IP and associate it with your Nginx Load Balancer server.

    _Screenshot placeholder for Elastic IP allocation_

2. **Point FreeDNS subdomain to your Elastic IP:**
   - Go to the DNS section in FreeDNS and add an **A Record** that points to your Elastic IP.

    _Screenshot placeholder for DNS settings in FreeDNS_

## <a name="configure-nginx"></a> 5. Configure Nginx to Recognize the Domain

1. **SSH into your Nginx server:**

    ```bash
    ssh -i your-key.pem ubuntu@your-elastic-ip
    ```

2. **Update Nginx to recognize your domain:**

    Open the `nginx.conf` file again:

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

    Update the `server_name` field with your subdomain:

    ```nginx
    server_name loadbalancernginx.chickenkiller.com;
    ```

    _Screenshot placeholder for updated Nginx config with domain_

3. **Test and reload Nginx:**

    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```

    _Screenshot placeholder for Nginx test and reload_

## <a name="install-certbot"></a> 6. Install SSL Certificate with Certbot

1. **Ensure the `snapd` service is running:**

    ```bash
    sudo systemctl status snapd
    ```

    _Screenshot placeholder for snapd status_

2. **Install Certbot:**

    ```bash
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```

    _Screenshot placeholder for Certbot installation_

3. **Request SSL certificate:**

    Run Certbot to generate and install your SSL certificate:

    ```bash
    sudo certbot --nginx
    ```

    _Screenshot placeholder for Certbot running and successful certificate issue_

4. **Test HTTPS access:**

    Visit your site using HTTPS:

    ```bash
    https://loadbalancernginx.chickenkiller.com
    ```

    You should see the padlock icon, indicating a secure connection.

    _Screenshot placeholder for padlock and HTTPS confirmation_

## <a name="automatic-renewal"></a> 7. Set Up Automatic Renewal of SSL Certificates

1. **Test the renewal process:**

    ```bash
    sudo certbot renew --dry-run
    ```

    _Screenshot placeholder for dry-run test_

2. **Set up a cron job to automatically renew the SSL certificate:**

    Edit your crontab file:

    ```bash
    crontab -e
    ```

    Add the following line to renew the certificate twice daily:

    ```bash
    * */12 * * * /usr/bin/certbot renew > /dev/null 2>&1
    ```

    _Screenshot placeholder for crontab setup_

## <a name="conclusion"></a> 8. Conclusion

Congratulations! You have successfully configured an Nginx Load Balancer with SSL/TLS using a FreeDNS subdomain and AWS Elastic IP. You’ve also set up automatic SSL renewal to ensure your connection remains secure.

---

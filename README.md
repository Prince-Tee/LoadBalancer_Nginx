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

First, let us rename our Apache load balancer server on AWS 

![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/rename%20the%20apache%20server%20on%20aws.png)

open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections

![SCREENSHOT](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/open%20port%20443%20and%2080.png)

Now Lets uninstall Apache from the existing Load Balancer server setting up Nginx:

1. **SSH into your server:**

    ```bash
    ssh -i <your pem file.pem> ubuntu@your-server-ip
    ```
![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/ssh%20into%20the%20server.png)

2. **Stop the Apache service:**

    ```bash
    sudo systemctl stop apache2
    ```

3. **Uninstall Apache:**

    ```bash
    sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
    ```

    ![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/remove%20apache%20from%20the%20existing%20server.png)

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

    ![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/remove%20apache%20from%20the%20existing%20server.png)

 Open the /etc/hosts file: Use a text editor like nano or vi to open the /etc/hosts file for editing. You need root privileges to modify this file.

```bash
sudo vi /etc/hosts
```

Add entries for Web1 and Web2: In the /etc/hosts file, add the local IP addresses of your web servers and their corresponding names. The format is:

```plaintext

<IP_address>   <hostname>
```
![screesnhot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/verify%20the%20content%20of%20%20sudo%20vi%20etc%20hosts%20still%20points%20to%20web%201%20and%20web2.png)   

## <a name="install-nginx"></a> 2. Install and Configure Nginx as Load Balancer

1. **Update your system and install Nginx:**

    ```bash
    sudo apt update
    sudo apt install nginx
    ```

    ![_Screenshot placeholder for Nginx installation step_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/update%20and%20install%20nginx.png)

2. **Open the Nginx configuration file:**

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

    ![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/sudo%20vi%20etcnginxnginx.conf.png)

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

    ![_Screenshot placeholder for the Nginx config update_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/update%20the%20sudo%20vi%20etcnginxnginx.conf%20and%20comment%20out%20.png)

4. **Comment out the following line:**

    ```bash
    #       include /etc/nginx/sites-enabled/*;
    ```

    ![_Screenshot placeholder for the Nginx config update_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/update%20the%20sudo%20vi%20etcnginxnginx.conf%20and%20comment%20out%20.png)

5. **Restart Nginx:**

    ```bash
    sudo systemctl restart nginx
    sudo systemctl status nginx
    ```

    ![_Screenshot placeholder for Nginx status check_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/Restart%20Nginx.png)

## <a name="register-freedns"></a> 3. Register and Use FreeDNS Subdomain

1. **Register a subdomain:**
   - Go to [FreeDNS](https://freedns.afraid.org) and register.
   - Choose a subdomain (e.g., `yourname.mooo.com`).


## <a name="assign-elastic-ip"></a> 4. Assign Elastic IP and Point FreeDNS Subdomain

1. **Allocate Elastic IP:**
   - In the AWS EC2 dashboard, click on **Elastic IPs**.
   - Allocate an IP and associate it with your Nginx Load Balancer server.

    _Screenshot placeholder for Elastic IP allocation_

2. **Point FreeDNS subdomain to your Elastic IP:**
   - Go to the DNS section in FreeDNS and add an **A Record** that points to your Elastic IP.

    ![_Screenshot placeholder for DNS settings in FreeDNS_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/adding%20your%20elastic%20ip%20to%20subdomain.PNG)

Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol - http://<your-domain-name.com>
in our case I have this
http://loadbalancernginx.chickenkiller.com

![screesnhot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/checking%20that%20our%20subdomain%20can%20serve%20ou%20webservers.PNG)


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

    ![_Screenshot placeholder for updated Nginx config with domain_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/specify%20the%20server%20name%20of%20our%20subdomain%20in%20nginc%20conf.PNG)

3. **Test and reload Nginx:**

    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```

    ![_Screenshot placeholder for Nginx test and reload_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/test%20that%20nginx%20is%20sucessful.PNG)

## <a name="install-certbot"></a> 6. Install SSL Certificate with Certbot

1. **Ensure the `snapd` service is running:**

    ```bash
    sudo systemctl status snapd
    ```

    ![_Screenshot placeholder for snapd status_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/sudo%20systemctl%20status%20snapd.PNG)

2. **Install Certbot:**

    ```bash
    sudo snap install --classic certbot
    ```

    ![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/install%20certbot%20on%20ngix%20server.PNG)

    ```bash
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```

   ![ _Screenshot placeholder for Certbot installation_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/reqsting%20https%20from%20certbot%20for%20our%20subdomain.PNG)

3. **Request SSL certificate:**

    Run Certbot to generate and install your SSL certificate:

    ```bash
    sudo certbot --nginx
    ```

    ![_Screenshot placeholder for Certbot running and successful certificate issue_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/reqsting%20https%20from%20certbot%20for%20our%20subdomain.PNG)

4. **Test HTTPS access:**

    Visit your site using HTTPS:

    ```bash
    https://loadbalancernginx.chickenkiller.com
    ```

    You should see the padlock icon, indicating a secure connection.

    ![_Screenshot placeholder for padlock and HTTPS confirmation_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/our%20site%20certicate%20shown.PNG)

## <a name="automatic-renewal"></a> 7. Set Up Automatic Renewal of SSL Certificates

1. **Test the renewal process:**

    ```bash
    sudo certbot renew --dry-run
    ```

    ![_Screenshot placeholder for dry-run test_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/test%20renewal%20command%20in%20dry-run%20mode.PNG)

2. **Set up a cron job to automatically renew the SSL certificate:**

    Edit your crontab file:

    ```bash
    crontab -e
    ```
    ![screenshot](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/crontab%20e.PNG)

    You'll see a list of available editors. Since we are looking for the easiest option, choose 1 for nano (the easiest text editor to use):

      ```bash
      Choose 1-4 [1]: 1
       ```

    Add the following line to renew the certificate twice daily:

    ```bash
    * */12 * * * /usr/bin/certbot renew > /dev/null 2>&1
    ```

    ![_Screenshot placeholder for crontab setup_](https://github.com/Prince-Tee/LoadBalancer_Nginx/blob/main/screenshot%20from%20my%20local%20env/add%20the%20to%20let%20cron%20renew%20cet%20every%202%20day.PNG)

## <a name="conclusion"></a> 8. Conclusion

Congratulations! You have successfully configured an Nginx Load Balancer with SSL/TLS using a FreeDNS subdomain and AWS Elastic IP. You’ve also set up automatic SSL renewal to ensure your connection remains secure.

---

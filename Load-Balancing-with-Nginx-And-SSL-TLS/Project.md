# Load Balancing Solution with Nginx and SSL/TLS

This project is a continuation of the `Tooling Website Solution`, it will guide you through configuring `Nginx` as a load balancer and securing your website with `SSL/TLS` certificates using `LetEncrypt`. We'll achieve this in two parts:

1. Configuring Nginx as a Load Balancer.
2. Registering a domain name and configuring a secured connection using SSL/TLS certificates.

## Part 1: Configure Nginx as a Load Balancer

### Step 1: Create an EC2 Instance
- **Launch an EC2 instance** on AWS based on Ubuntu 24.04 LTS and name it `Nginx LB`.
    ![Instance List](./images/ec2-instance-list.PNG)

- **Security Group Configuration**:
  - Open TCP port 80 for HTTP.
  - Open TCP port 443 for HTTPS.
   ![Security Group](./images/nginx-LB-SG.PNG)

### Step 2: Update `/etc/hosts` File
- Connect to your EC2 instance via SSH.
    ```sh
    ssh -i keypair.pem ubuntu@server-public-ip
    ```

- Update the `/etc/hosts` file with the names and IP addresses of your web servers:
    ```sh
    sudo vi /etc/hosts
    ```

    - Add the following lines:
        ```
        <web-server-private-ip> Web1
        <web-server-private-ip> Web2
        ```
    ![Hosts File](./images/modifiy-etc-hosts.PNG)

### Step 3: Install and Configure Nginx
- Update and upgrade the system:
    ```
    sudo apt update && sudo apt upgrade -y
    ```

- Install Nginx:
    ```
    sudo apt install nginx -y
    ```

### Step 4: Configure Nginx as a Load Balancer
- Open the Nginx configuration file:
    ```
    sudo vi /etc/nginx/nginx.conf
    ```

- Insert the following configuration into the `http` section:
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

    # Comment out this line
    # include /etc/nginx/sites-enabled/*;
    ```
    ![Configure Nginx](./images/configure-nginx-as-LB.PNG)

- Restart Nginx and ensure the service is running:
    ```sh
    sudo nginx -t
    sudo systemctl restart nginx
    sudo systemctl status nginx
    ```
    ![Nginx Service Active](./images/verify-nginx-config-file-syntax.PNG)

## Part 2: Register a Domain and Configure SSL/TLS

### Step 1: Register a Domain Name
- Register a new domain name using any domain registrar (e.g., GoDaddy, Domain.com, Bluehost), but for this implementation i used AWS Cloud service `Route53` also built for this purpose. I already have a domain name hosted in `Route53` named `mightygull.click`.
    
    - Accessing `Route53` Dashboard on AWS:
     ![Route53 Dashboard](./images/create-a-record-01.PNG)

     - Hosted Zone:
      ![Hosted Zones](./images/create-a-record-02.PNG)

### Step 2: Create & Assign Elastic IP and Update DNS
- To create an `Elastic IP` navigate to `VPC` services on the AWS management console and on the left side of the management console under `Virtual Private Cloud` click `Elastic IP` > `Allocate Elastic IP address`.
- Assign the Elastic IP to your `Nginx LB` EC2 instance.
    - Associate the `Elastic IP` to Nginx EC2-Instance:
     ![Associate EIP 1](./images/associate-elastic-ip-01.PNG)
     ![Associate EIP 2](./images/associate-elastic-ip-02.PNG)
     ![Associate EIP 3](./images/associate-elastic-ip-03.PNG)

- Update the `A-Record` in your domain registrar to point to the Elastic IP of your `Nginx LB` instance.
 ![Creating A-Record Set](./images/create-a-record-03.PNG)
 ![Pointing Elastic IP To NginxLB](./images/create-a-record-04.PNG)
 ![Verify A-Record INSYNC](./images/verify-a-record-is-ready-to-direct-traffic-to-elastic-ip.PNG)
 ![A-Record Created](./images/a-record-created-successfully.PNG)

### Step 3: Configure Nginx for Your Domain
- Update the `nginx.conf` file to recognize your new domain name:
    ```sh
    sudo vi /etc/nginx/nginx.conf
    ```
    Replace `server_name www.domain.com` with `server_name www.<your-domain-name.com>`.

    ![Nginx Config File](./images/update-nginx-config-file.PNG)

### Step 4: Install Certbot and Obtain SSL Certificate
- Ensure `snapd` service is active:
    ```sh
    sudo systemctl status snapd
    ```
    ![Verify snapd](./images/ensure-snapd-service-is-running.PNG)

- Install Certbot:
    ```sh
    sudo snap install --classic certbot
    ```
    ![Install Certbot](./images/install-certbot.PNG)

- Create a symlink for Certbot:
    ```sh
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```

- Obtain and install the SSL certificate:
    ```sh
    sudo certbot --nginx
    ```
    ![Certificate Request](./images/certificates-request-certbot.PNG)

### Step 5: Test Secure Access
- Verify that you can access your website securely by navigating to `www.<your-domain-name.com>` in your browser.

### Step 6: Set Up Certificate Renewal
- Test renewal in dry-run mode:
    ```sh
    sudo certbot renew --dry-run
    ```
    ![Certbot Dry-Run](./images/test-cretificate-renewal-dry-run.PNG)

- Configure a cron job to renew the certificate periodically:
    ```sh
    crontab -e
    ```
    ![Create Cronjob](./images/input-cronjob-in-crontab.PNG)

    > N.B: Select an editor to modify the file (e.g Input 1-4, where `1` is nano).

- Add the following line to the crontab file:
    ```javascript
    * */12 * * * root /usr/bin/certbot renew > /dev/null 2>&1
    ```
    ![Input Cronjob](./images/input-cronjob-in-crontab-01.PNG)

- Verify the `Cronjob` configuration was successful
    ```
    crontab -l
    ```
    ![Verify CronJob](./images/verify-cronjob-input.PNG)

- Accessing the Tooling Website using the domain `www.mightygull.click`.

    ![Certificate Details](./images/view-certificate-issuer.PNG)
    ![Domain Webpage 1](./images/web-servers-accessable-from-domain-name.PNG)
    ![Domain Webpage 2](./images/web-servers-accessable-from-domain-name-01.PNG)

- Also the `Access logs` of both Web Servers
    ```sh
    sudo tail -f /var/log/httpd/access_log
    ```
    - Web Server 1:
     ![Access Log 1](./images/access-lob-web-1.PNG)

    - Web Server 2:
     ![Access Log 2](./images/access-lob-web-2.PNG)


### Conclusion

Congratulations! You have successfully configured an `Nginx load balancer with SSL/TLS` secured connection and set up automatic certificate renewal. Your web solution is now load-balanced and secured with HTTPS.

---

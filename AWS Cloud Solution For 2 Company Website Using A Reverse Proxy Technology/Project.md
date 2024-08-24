# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

## Starting Your AWS Cloud Project

### Requirements Before You Begin

1. **Configure Your AWS Account and Organization Unit:**

   - **Create an AWS Master Account (Root Account):**
     - Sign up for AWS with a primary email to create your root account.
   
   - **Create a Sub-Account and Name it `DevOps`:**
     - Under the root account, create a sub-account named "DevOps". Note that you will need a different email address to set up this sub-account.
      ![Create Root Account](./images/create-ou-01.PNG)
   
   - **Create an AWS Organization Unit (OU) Named `Dev`:**
     - In the root account, navigate to the AWS Organizations service and create an Organizational Unit (OU) named "Dev". This will be where all resources related to development are managed.
      ![Create OU](./images/create-ou-02.PNG)
   
   - **Move the DevOps Account into the Dev OU:**
     - Once the Dev OU is created, move the DevOps sub-account into the Dev OU to ensure proper resource and policy management.
      
       **Output 1:**
       ![Move OU](./images/move-account-ou.PNG)

       **Output 2:**
       ![Move OU](./images/move-account-ou-01.PNG)

       **Output 3:**
       ![Move OU](./images/move-account-ou-02.PNG)
   
   - **Log In to the Newly Created AWS DevOps Account:**
     - Use the email associated with the DevOps sub-account to log in.
      ![Login](./images/accessing-aws-mgt-console.PNG)

2. **Register a Free Domain Name for Your Fictitious Company:**

   - Use the `Cloud DNS` domain registrar to register a free domain name for your company, in this case i choose `Devtank`.
    ![Create Domain](./images/create-free-domain.PNG)

3. **Create a Hosted Zone in AWS Route 53 and Map it to Your Free Domain:**

   - Navigate to Route 53 in the AWS Management Console.
   - Create a new hosted zone using the domain name you registered from Cloud DNS.
    
     **Output 1:**
     ![Create HZ](./images/create-hosted-zone-01.PNG)

     **Output 2:**
     ![Create HZ](./images/create-hosted-zone-02.PNG)

### Setting Up a Virtual Private Network (VPC)

1. **Create a VPC:**
   - Go to the VPC service in the AWS Management Console.
   - Click on "Create VPC" and set up a VPC for your project.
    ![VPC](./images/create-vpc-01.PNG)

2. **Create Subnets:**
   - Create subnets as per your architecture requirements, ensuring you have both public and private subnets.
    ![Subnets](./images/final-subnet-creation.PNG)

3. **Create Route Tables and Associate Them:**

   - **Public Subnets:**
     - Create a route table specifically for public subnets.
      ![Public-RT](./images/create-public-rtb-01.PNG)

     - Associate this route table with the public subnets.
      ![Public-RT](./images/create-public-rtb-03.PNG)

   - **Private Subnets:**
     - Create another route table for private subnets.
      ![Private-RT](./images/create-private-rtb-02.PNG)

     - Associate it with the private subnets.
      ![Private-RT](./images/create-private-rtb-01.PNG)

4. **Create an Internet Gateway:**
   - Attach an Internet Gateway to your VPC to allow internet access for your public subnets.
    ![IGW](./images/create-igw.PNG)

5. **Configure Public Route Tables for Internet Access:**
   - Edit the route in the public route table to associate it with the Internet Gateway. This allows public subnets to be accessible from the Internet.
    ![Edit Public-RT](./images/config-public-rt.PNG)

6. **Create Elastic IPs:**
   - Go to the Elastic IPs section and create 3 Elastic IP addresses.

7. **Set Up a NAT Gateway:**
   - Create a NAT Gateway in one of the public subnets and assign one of the Elastic IPs to it. The NAT Gateway will enable internet access for instances in the private subnets.
     ![NAT-GW](./images/create-nat-gw.PNG)

    - Configure Private Route Tables for Internet Access
     ![RT](./images/config-private-rt.PNG)

8. **Create Security Groups:**

   - **Nginx Servers:**
     - Create a security group for Nginx servers. Which will allow access only from the Application Load Balancer (ALB).
      ![Nginx-SG](./images/nginx-sg.PNG)

   - **Bastion Servers:**
     - Create a security group for Bastion servers, allowing SSH access only from your workstation’s public IP address.
      ![Bastion-SG](./images/bastion-sg.PNG)

   - **External & Internal Application Load Balancer (ALB):**
     - Set up a security group for the External ALB to allow traffic from the internet.
      ![External-ALB-SG](./images/external-alb-sg.PNG)
 
     - Set up a security group for Internal ALB to allow traffic from only Nginx.
      ![Internal-ALB-SG](./images/internal-alb-sg.PNG)

   - **Web Servers:**
     - Create a security group for web servers to allow access only from the Internal ALB.
      ![Webservers-SG](./images/webservers-sg.PNG)

   - **Data Layer:**
     - Create a security group for the data layer, allowing access to RDS only from web servers and to EFS mount points from both Nginx and web servers.
      ![Data-SG](./images/data-layer-sg.PNG)

### AWS Key Management Service (KMS) Setup

1. **Create a Customer Managed Key (CMK):**
   - Navigate to the AWS KMS service.
   - Choose to create a new symmetric encryption CMK.
   - Provide an alias (e.g., `my-app-key`) for the CMK.
   - Set the key usage permissions by selecting AWS services and IAM users/roles that are allowed to use the key.

2. **Configure Key Policies:**
   - Set key policies to define who can administer and use the CMK.
   - Include policies for IAM users, roles, and AWS services that need to encrypt or decrypt data.
     
     **Output 1:**
     ![KMS](./images/create-kms-01.PNG)

     **Output 2:**
     ![KMS](./images/create-kms-02.PNG)

     **Output 3:**
     ![KMS](./images/create-kms-03.PNG)

     **Output 4:**
     ![KMS](./images/create-kms-04.PNG)

     **Output 5:**
     ![KMS](./images/create-kms-05.PNG)

     **Output 6:**
     ![KMS](./images/create-kms-06.PNG)

### AWS Certificate Manager (ACM) Setup

1. **Request a Public Certificate:**
   - Navigate to the AWS Certificate Manager service.
   - Request a new public SSL/TLS certificate for your domain (e.g., `*.yourdomain.com`).
   - Choose DNS validation as the validation method.

2. **Validate Your Domain Ownership:**
   - ACM will provide a CNAME record that you need to add to your DNS provider (e.g., Route 53) to prove domain ownership.
   - Once the CNAME record has been created successfully in the Route 53 hosted zone, we will need to configure the CNAME record in the `Cloudns.net` domain. This step will validate and issue the certificate.
     
     **Output 1:**
     ![ACM](./images/request-new-cert-01.PNG)

     **Output 2:**
     ![ACM](./images/request-new-cert-02.PNG)

     **Output 3:**
     ![ACM](./images/request-new-cert-03.PNG)

     **Output 4:**
     ![ACM](./images/validating-crt-domain-01.PNG)

     **Output 5:**
     ![ACM](./images/validating-crt-domain-02.PNG)

     **Output 6:**
     ![ACM](./images/validating-crt-domain-03.PNG)

     **Output 7:**
     ![ACM](./images/validating-crt-domain-04.PNG)

     **Output 8:**
     ![ACM](./images/validating-crt-domain-05.PNG)

### Set Up Amazon Elastic File System (EFS)

1. **Create an EFS for Shared File Storage:**
   - Navigate to the Amazon EFS service and create a new file system.
   - Configure it to be accessible by both Nginx and web server instances.

2. **Set Up Mount Targets in Each Subnet:**
   - Ensure you create mount targets for EFS in each subnet that houses an EC2 instance.

3. **Create EFS Access Points:**
   - This will specify where the webservers will mount with, thus creating 2 mount points for `Tooling` and `Wordpress` servers each.
     
     **Output 1:**
     ![EFS](./images/create-efs-01.PNG)

     **Output 2:**
     ![EFS](./images/create-efs-02.PNG)

     **Output 3:**
     ![EFS](./images/create-efs-03.PNG)

     **Output 4:**
     ![EFS](./images/create-access-point-01.PNG)

     **Output 5:**
     ![EFS](./images/create-access-point-02.PNG)

     **Output 6:**
     ![EFS](./images/create-access-point-03.PNG)

     **Output 7:**
     ![EFS](./images/create-access-point-04.PNG)

### Set Up Amazon Relational Database Service (RDS)

> Note: First create a subnet group in the private subnets as required in the project architechture for your RDS before going ahead to create an RDS instance.

1. **Create an RDS Instance:**
   - Use AWS RDS service to set up a MySQL database.
   - Ensure the database is deployed in the private subnets within your VPC.

2. **Configure RDS Security and Networking:**
   - Set the appropriate security group to allow access only from web server instances.
     
     **Output 1:**
     ![RDS](./images/create-rds-01.PNG)

     **Output 2:**
     ![RDS](./images/create-rds-02.PNG)

     **Output 3:**
     ![RDS](./images/create-rds-03.PNG)

     **Output 4:**
     ![RDS](./images/create-rds-04.PNG)

     **Output 5:**
     ![RDS](./images/create-rds-05.PNG)

     **Output 6:**
     ![RDS](./images/create-rds-06.PNG)

     **Output 7:**
     ![RDS](./images/create-rds-07.PNG)

     **Output 8:**
     ![RDS](./images/create-rds-08.PNG)

     **Output 9:**
     ![RDS](./images/create-rds-09.PNG)

     **Output 10:**
     ![RDS](./images/create-rds-010.PNG)

     **Output 11:**
     ![RDS](./images/create-rds-011.PNG)

     **Output 12:**
     ![RDS](./images/create-rds-012.PNG)

### Configure Application Load Balancers (ALB)

> Note: Before creating the ALBs, we have to create the target groups first, you can find guidelines on how to do this in the `Setting Up Compute Resources Section`.

![List-TG](./images/target-groups-created.PNG)

1. **Create an Internet-Facing ALB:**
   - Configure the ALB to listen on the HTTPS protocol (TCP port 443).
   - Choose the appropriate VPC, AZ, and subnets for the ALB.
   - Select the TLS certificate from ACM.
   - Assign the correct security group to the ALB.
   - Configure the ALB with the Nginx target group.
     
     **Output 1:**
     ![ALB](./images/create-ALB-01.PNG)

     **Output 2:**
     ![ALB](./images/create-ALB-02.PNG)

     **Output 3:**
     ![ALB](./images/create-ALB-03.PNG)

     **Output 4:**
     ![ALB](./images/create-ALB-04.PNG)

     **Output 5:**
     ![ALB](./images/create-ALB-05.PNG)

2. **Create an Internal ALB for Web Servers:**
   - Set the ALB to listen on HTTPS (TCP port 443).
   - Choose the appropriate VPC, AZ, and subnets for the ALB.
   - Select the TLS certificate from ACM.
   - Assign the correct security group to the ALB.
   - Use the appropriate target group for routing traffic to the web servers, in this case we have the `Wordpress` target group and `Tooling` target group.
    
     **Output:**
     ![ALB-List](./images/list-ALB.PNG)
   
   > Note: The default target group configured on the listener while creating the internal load balancer is to forward traffic to `Wordpress` on port 443. Hence, we need to create a rule to route traffic to `Tooling` as well.

   - Choose the load balancer where you want to add the rule.
   - Listeners Tab:
     - Click on the Listeners tab.
     - Select the listener (HTTPS:443) and click `Manage listener`.
   - Click on Add rule.
   - Configure the Rule:
     - Give the rule a name and click next.
     - Add a condition by selecting `Host header`.
     - Enter the hostnames for which you want to route traffic. (e.g. `tooling.com` and `www.tooling.com`).
     - Choose the appropriate target group for the hostname.
       
       **Output 1:**
       ![ALB-Rule](./images/add-tooling-rule.PNG)

       **Output 2:**
       ![ALB-Rule](./images/add-tooling-rule-01.PNG)

       **Output 3:**
       ![ALB-Rule](./images/add-tooling-rule-02.PNG)

       **Output 4:**
       ![ALB-Rule](./images/add-tooling-rule-03.PNG)

       **Output 5:**
       ![ALB-Rule](./images/add-tooling-rule-04.PNG)

### Setting Up Compute Resources

You will need to set up and configure several compute resources within your VPC. The key components related to compute resources are:

- **EC2 Instances**
- **Launch Templates**
- **Target Groups**
- **Auto Scaling Groups**
- **TLS Certificates**
- **Application Load Balancers (ALB)**

#### Setting Up Compute Resources for Nginx

1. **Create EC2 Instances for Nginx:**
   - Launch a RedHat Linux EC2 instance in two different Availability Zones (AZ) within your selected AWS region (preferably the region closest to your customers).
   - Use a T3 family instance type (e.g., t3.small).

2. **Install Necessary Software on EC2 Instances:**
   - Ensure the following software is installed:
     - Python, chrony, net-tools, vim, wget, telnet, epel-release, htop.
     - Below is the script to get the Nginx AMI ready.
 
        ```bash
        sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
        sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
        sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
        sudo systemctl start chronyd
        sudo systemctl enable chronyd
        sudo systemctl status chronyd
       
        # Set SELinux policies so that our servers can function properly on all the redhat instance
        sudo setsebool -P httpd_can_network_connect=1
        sudo setsebool -P httpd_can_network_connect_db=1
        sudo setsebool -P httpd_execmem=1
        sudo setsebool -P httpd_use_nfs 1

        # Install Amazon EFS utils for mounting the targets on the Elastic file system
        git clone https://github.com/aws/efs-utils
        cd efs-utils

        sudo yum install -y make
        sudo yum install -y rpm-build
        sudo yum install openssl-devel -y
        sudo yum install cargo -y
        sudo make rpm
        sudo yum install -y  ./build/amazon-efs-utils*rpm

        # Set up self-signed certificate for the nginx instance
        sudo mkdir /etc/ssl/private
        sudo chmod 700 /etc/ssl/private
        sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/devtank.key -out /etc/ssl/certs/devtank.crt
        sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
        ```

3. **Create an AMI from the Configured EC2 Instance:**
   - Create an Amazon Machine Image (AMI) from the EC2 instance after installing the required software.
     
     **Output 1:**
     ![Nginx-AMI](./images/create-nginx-ami-01.PNG)

     **Output 2:**
     ![Nginx-AMI](./images/create-nginx-ami.PNG)

4. **Prepare a Launch Template for Nginx:**
   - Use the created AMI to set up a launch template for Nginx.
   - Ensure that the instances launched from this template are placed in public subnets and assigned the appropriate security group.
   - Add user data to update the yum package repository and install Nginx upon instance launch.
     
     **Output 1:**
     ![Nginx-LT](./images/create-nginx-lt-01.PNG)

     **Output 2:**
     ![Nginx-LT](./images/create-nginx-lt-02.PNG)

     **Output 3:**
     ![Nginx-LT](./images/create-nginx-lt-03.PNG)

     **Output 4:**
     ![Nginx-LT](./images/create-nginx-lt-04.PNG)

     **User-Data:**
     - Link to the User-Data can be found in this [link](https://github.com/alagbaski/devtank_user_data_config/tree/main).

5. **Create Target Group for Nginx:**
   - Set the target type as "Instances".
   - Use the HTTPS protocol on the secure TLS port (443).
   - Set the health check path to `/healthstatus` and register the Nginx instances as targets.
     
     **Output 1:**
     ![Nginx-TG](./images/config-nginx-tg-01.PNG)

     **Output 2:**
     ![Nginx-TG](./images/config-nginx-tg-02.PNG)

     **Output 3:**
     ![Nginx-TG](./images/config-nginx-tg-03.PNG)

     **Nginx Health Status:**
     ![Nginx](./images/nginx-health-status.PNG)

6. **Configure Auto Scaling for Nginx:**
   - Select the right launch template and VPC.
   - Choose both public subnets for deployment.
   - Enable the ALB for the Auto Scaling Group (ASG) and select the created target group.
   - Set health checks for both EC2 instances and the ALB.
   - Define desired capacity (2), minimum capacity (2), and maximum capacity (4).
   - Set scaling policies to scale out if CPU utilization reaches 90%.
   - Set up an SNS topic to receive scaling notifications.
     
     **Output 1:**
     ![Nginx-ASG](./images/config-nginx-asg-01.PNG)

     **Output 2:**
     ![Nginx-ASG](./images/config-nginx-asg-02.PNG)

     **Output 3:**
     ![Nginx-ASG](./images/config-nginx-asg-03.PNG)

     **Output 4:**
     ![Nginx-ASG](./images/config-nginx-asg-04.PNG)

     **Output 5:**
     ![Nginx-ASG](./images/config-nginx-asg-05.PNG)

#### Setting Up Compute Resources for Bastion Hosts

1. **Create EC2 Instances for Bastion Hosts:**
   - Launch an RHEL EC2 instance in each AZ where Nginx servers were created.

2. **Install Required Software on Bastion Instances:**
   - Install the following software:
     - Python, chrony, net-tools, vim, wget, telnet, epel-release, htop.
     - Below is the script to get the Bastion AMI ready.
      
       ```bash
       sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
       sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
       sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
       sudo systemctl start chronyd
       sudo systemctl enable chronyd
       sudo systemctl status chronyd
       ```

3. **Associate Elastic IPs with Bastion Instances:**
   - Associate an Elastic IP with each Bastion EC2 instance.

4. **Create an AMI from the Bastion Instance:**
   - After setting up the required software, create an AMI from the Bastion instance.

5. **Prepare a Launch Template for Bastion Hosts:**
   - Use the created AMI to set up a launch template for Bastion hosts.
   - Launch instances in public subnets and assign the appropriate security group.
   - Configure user data to update the yum package repository and install Ansible and Git.
     
     **User-Data:**
     - Link to the User-Data can be found in this [link](https://github.com/alagbaski/devtank_user_data_config/tree/main).

#### Provision EC2 Instances for Web Servers

1. **Create EC2 Instances for Web Servers:**
   - Launch an RHEL EC2 instance each for WordPress and Tooling websites in each AZ.

2. **Install Necessary Software on Web Server Instances:**
   - Install the following software:
     - Python, chrony, net-tools, vim, wget, telnet, epel-release, htop, php.
     - Below is the script used in creating the Web Servers AMI.
       
       ```bash
       sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
       sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
       sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
       sudo systemctl start chronyd
       sudo systemctl enable chronyd
       sudo systemctl status chronyd
       
       # Set SELinux policies so that our servers can function properly on all the redhat instance
       sudo setsebool -P httpd_can_network_connect=1
       sudo setsebool -P httpd_can_network_connect_db=1
       sudo setsebool -P httpd_execmem=1
       sudo setsebool -P httpd_use_nfs 1

       # Install Amazon EFS utils for mounting the targets on the Elastic file system
       git clone https://github.com/aws/efs-utils
       cd efs-utils

       sudo yum install -y make
       sudo yum install -y rpm-build
       sudo yum install openssl-devel -y
       sudo yum install cargo -y
       sudo make rpm
       sudo yum install -y  ./build/amazon-efs-utils*rpm

       # Set up self-signed certificate for the webserver instance
       sudo yum install -y mod_ssl
       sudo openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/devtank.key -x509 -days 365 -out /etc/pki/tls/certs/devtank.crt
       ```
       ```bash
       # Edit the ssl.conf to confirm with the key and crt file created.
       sudo vim /etc/httpd/conf.d/ssl.conf
       ```
       ![Webserver-Self-CRT](./images/self-assigned-crt-webserver.PNG)

3. **Create an AMI from the Web Server Instance:**
   - After installing the necessary software, create an AMI from the web server instance, just like we did for Nginx and Bastion.

4. **Prepare Launch Templates for Web Servers:**
   - Use the created AMI to set up separate launch templates for WordPress and Tooling web servers.
   - Ensure instances are launched in private subnets and assign the appropriate security group.
   - Configure user data to update the yum package repository and install `WordPress` (only for WordPress launch template). Also for Tooling configure the user data to clone the `Tooling` code repository and carry out the necessary configurations to deploy the Tooling website.
   
   > Link to both user data can be found in this [link](https://github.com/alagbaski/devtank_user_data_config/tree/main).

5. **Configure Target Groups for Web Servers:**
   - Set the target type as "Instances".
   - Use the HTTPS protocol on the secure TLS port (443).
   - Set the health check path to `/healthstatus` and register the Web Server instances as targets.
     
     **Wordpress Health Status:**
     ![Wordpress](./images/Wordpress-health-status.PNG)

     **Tooling Health Status:**
     ![Tooling](./images/tooling-healthh-status.PNG)

6. **Configure Auto Scaling for Web Servers:**
   - Select the right launch template and VPC.
   - Choose both private subnets for deployment.
   - Enable the ALB for the Auto Scaling Group (ASG) and select the created target group.
   - Set health checks for both EC2 instances and the ALB.
   - Define desired capacity (2), minimum capacity (2), and maximum capacity (4).
   - Set scaling policies to scale out if CPU utilization reaches 90%.
   - Set up an SNS topic to receive scaling notifications.

### Configure DNS with Route 53

- **Create DNS Records for Services:**

    Earlier in this project we registered a free domain with `Cloudns` and configured a hosted zone in `Route53`. But that is not all that needs to be done as far as DNS configuration is concerned.

    We need to ensure that the main domain for the `WordPress` website can be reached, and the subdomain for `Tooling` website can also be reached using a browser.
     
     - Create an alias record for the root domain and direct its traffic to the ALB DNS name.
     - Create an alias record for `wordpress.devtank.cloudns.be` and `tooling.devtank.cloudns.be` and direct its traffic to the ALB DNS name.
     
    **Output 1:**
    ![A-Record](./images/create-A-record-for-wordpress.PNG)

    **Output 2:**
    ![A-Record](./images/create-A-record-for-tooling.PNG)

### Test your websites
- Now let us access our `wordpress` & `tooling` website via a browser using our DNS name.
  
  **Accessing WordPress:**
  ![Wordpress](./images/wordpress-website.PNG)

  **Accessing Tooling:**
  ![Tooling](./images/tooling-website.PNG)


## Summary

By following the steps outlined above, you will have set up a robust, scalable, and highly available cloud infrastructure on AWS. This setup includes networking, compute resources, security groups, load balancing, TLS certificates, file storage, and databases. Make sure to test the setup thoroughly and monitor performance and security regularly.

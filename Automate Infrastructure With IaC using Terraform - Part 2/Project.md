# Automate Infrastructure With IaC using Terraform - Part 2

Based on our knowledge from the previous project, we can continue creating AWS resources.

## Networking - Private Subnets & Best Practices

- Create 4 private subnets keeping in mind the following principles:
  
  - Make sure you use variables or `length()` function to determine the number of AZs.
  - Use variables and `cidrsubnet()` function to allocate `vpc_cidr` for subnets.
  - Keep variables and resources in separate files for better code structure and readability.
  - Tag all the resources you have created so far. Explore how to use `format()` and `count` functions to automatically tag subnets with its respective number.

    ```tf
    terraform {
    required_providers {
        aws = {
        source  = "hashicorp/aws"
        version = "5.66.0"
        }
    }
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block           = var.vpc_cidr
    enable_dns_support   = var.enable_dns_support
    enable_dns_hostnames = var.enable_dns_hostnames

    tags = merge(
        var.tags,
        {
        Name = "${var.name}-vpc"
        },
    )
    }


    # Get list of availability zones
    data "aws_availability_zones" "available" {
    state = "available"
    }

    # Create 2 Public Subnet
    resource "aws_subnet" "public" {
    count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

    tags = merge(
        var.tags,
        {
        Name = format("%s-PublicSubnet-%s", var.name, count.index + 1)
        },
    )
    }

    # Create 4 private subnets
    resource "aws_subnet" "private" {
    count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
    map_public_ip_on_launch = false
    availability_zone = element(
        [
        data.aws_availability_zones.available.names[0], #Use the first AZ for private subnets 1 and 3
        data.aws_availability_zones.available.names[1]  #Use the second AZ for private subnets 2 and 4
        ],
        count.index % 2
    )

    tags = merge(
        var.tags,
        {
        Name = format("%s-PrivateSubnet-%s", var.name, count.index + 1)
        },
    )
    }
    ```

> Note: You can add multiple tags as a default set. for example, in out `terraform.tfvars` file we can have default tags defined.

```tf
tags = {
  Environment      = "production"
  Owner-Email     = "oladiposamson34@gmail.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```

Now we can tag all our resources using the format below:

```tf
tags = merge(
    var.tags,
    {
      Name = "Name of the resource"
    },
  )
```

**Output 1:**
![VPC](./images/vpc.PNG)

**Output 2:**
![Subnet](./images/subnets.PNG)

## Internet Gateways & `format()` function

- Create an Internet Gateway in a separate Terraform file `igw.tf`.

    ```tf
    resource "aws_internet_gateway" "igw" {
    vpc_id = aws_vpc.main.id

    tags = merge(
        var.tags,
        {
        Name = format("%s-%s", aws_vpc.main.id, "Ops_IGW")
        }
    )
    }
    ```

    **Output:**
    ![IGW](./images/igw.PNG)

    Did you notice how we have used `format()` function to dynamically generate a unique name for this resource? The first part of the `%s` takes the interpolated value of `aws_vpc.main.id` while the second `%s` appends a literal string `Ops_IGW` and finally an exclamation mark is added in the end.

    If any of the resources being created is either using the `count` function or creating multiple resources using a loop, then a key-value pair that needs to be unique must be handled differently.

    For example, each of our subnets should have a unique name in the tag section. Without the `format()` function, this would not be possible. With the `format` function, each private subnet's tag will look like this.

    ```
    Name = PrivateSubnet-0
    Name = PrivateSubnet-1
    Name = PrivateSubnet-2
    ```

## NAT Gateways

- Create 1 NAT Gateways and 1 Elastic IP (EIP) address, now use similar approach to create the NAT Gateways in a new file called `nat.tf`.

    > Note: We need to create an Elastic IP for the NAT Gateway, and you can see the use of `depends_on` to indicate that the Internet Gateway resource must be available before this should be created.

    ```tf
    resource "aws_eip" "nat_eip" {
    domain     = "vpc"
    depends_on = [aws_internet_gateway.igw]

    tags = merge(
        var.tags,
        {
        Name = format("%s-EIP", var.name)
        },
    )
    }


    resource "aws_nat_gateway" "nat" {
    allocation_id = aws_eip.nat_eip.id
    subnet_id     = element(aws_subnet.public.*.id, 0)
    depends_on    = [aws_internet_gateway.igw]

    tags = merge(
        var.tags,
        {
        Name = format("%s-Nat", var.name)
        },
    )
    }
    ```
    
    **Output 1:**
    ![NAT](./images/natgateway.PNG)

    **Output 2:**
    ![EIP](./images/elastic-ip.PNG)

## AWS routes

- Create a file called `route.tf` and use it to create route tables for both public and private subnets, create the resources below and ensure they are properly tagged.
  
  - aws_route_table
  - aws_route
  - aws_route_table_association

    ```tf
    # create private route table
    resource "aws_route_table" "private-rtb" {
    vpc_id = aws_vpc.main.id

    tags = merge(
        var.tags,
        {
        Name = format("%s-Private-Route-Table", var.name)
        },
    )
    }

    # Create route for the private route table and attach the nat gateway
    resource "aws_route" "private-rtb-route" {
    route_table_id         = aws_route_table.private-rtb.id
    destination_cidr_block = "0.0.0.0/0"
    nat_gateway_id         = aws_nat_gateway.nat.id
    }

    # associate all private subnets to the private route table
    resource "aws_route_table_association" "private-subnets-assoc" {
    count          = length(aws_subnet.private[*].id)
    subnet_id      = element(aws_subnet.private[*].id, count.index)
    route_table_id = aws_route_table.private-rtb.id
    }

    # create route table for the public subnets
    resource "aws_route_table" "public-rtb" {
    vpc_id = aws_vpc.main.id

    tags = merge(
        var.tags,
        {
        Name = format("%s-Public-Route-Table", var.name)
        },
    )
    }

    # create route for the public route table and attach the internet gateway
    resource "aws_route" "public-rtb-route" {
    route_table_id         = aws_route_table.public-rtb.id
    destination_cidr_block = "0.0.0.0/0"
    gateway_id             = aws_internet_gateway.igw.id
    }

    # associate all public subnets to the public route table
    resource "aws_route_table_association" "public-subnets-assoc" {
    count          = length(aws_subnet.public[*].id)
    subnet_id      = element(aws_subnet.public[*].id, count.index)
    route_table_id = aws_route_table.public-rtb.id
    }
    ```
- Now if you run `terraform plan` and `terraform apply`, it will add the following resources to AWS in multi-az set up:
  - [x] - Our main vpc
  - [x] - 2 Public subnets
  - [x] - 4 Private subnets
  - [x] - 1 Internet Gateway
  - [x] - 1 NAT Gateway
  - [x] - 1 EIP
  - [x] - 2 Route tables

  **Public Route & Table**
  
  - Output 1:
  ![RTB](./images/public-rtb.PNG)

  - Output 2:
  ![RTB](./images/public-rtb-01.PNG)

  **Private Route & Table**

  - Output 1:
  ![RTB](./images/private-rtb.PNG)

  - Output 2:
  ![RTB](./images/private-rtb-01.PNG)


Now that we are done with the Networking part of AWS set-up, let us move on to Compute and Access Control configuration automation using Terraform.

## AWS Identity & Access Management

### IAM and Roles

We want to pass an IAM role on EC2 instances to give them access to some specific resources, so we need to do the following:

- Create `AssumeRole`
  
  Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you may not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access.

  Add the following code to a new file named `iam_roles.tf`.

    ```tf
    resource "aws_iam_role" "ec2_instance_role" {
    name = "ec2_instance_role"
    assume_role_policy = jsonencode({
        Version = "2012-10-17"
        Statement = [
        {
            Action = "sts:AssumeRole"
            Effect = "Allow"
            Sid    = ""
            Principal = {
            Service = "ec2.amazonaws.com"
            }
        },
        ]
    })

    tags = merge(
        var.tags,
        {
        Name = "aws assume role"
        },
    )
    }
    ```

  In this code, we are creating `AssumeRole` with `AssumeRole policy`. It grants to an entity, in our case it is an EC2, permissions to assume the role.

- Create `IAM policy` for this role

  This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform an action describe applied to EC2 instances:

    ```tf
    resource "aws_iam_policy" "policy" {
    name        = "ec2_instance_policy"
    description = "A test policy"
    policy = jsonencode({
        Version = "2012-10-17"
        Statement = [
        {
            Action = [
            "ec2:Describe*",
            ]
            Effect   = "Allow"
            Resource = "*"
        },
        ]

    })

    tags = merge(
        var.tags,
        {
        Name = "aws assume policy"
        },
    )

    }
    ```

- Attach the `Policy` to the `IAM Role`

  This is where, we will be attaching the policy which we created above, to the role we created in the first step.

    ```tf
    resource "aws_iam_role_policy_attachment" "test-attach" {
    role       = aws_iam_role.ec2_instance_role.name
    policy_arn = aws_iam_policy.policy.arn
    }
    ```

- Create an `Instance Profile` and interpolate the `IAM Role`

    ```tf
    resource "aws_iam_instance_profile" "ip" {
    name = "aws_instance_profile_test"
    role = aws_iam_role.ec2_instance_role.name
    }
    ```

    We are pretty much done with the Identity and Management part for now, let us move on and create other resources required.

## Resources To Be Created

Going by our architecture, we need to do the following:

1. Create Security Groups.
2. Create Target Group for Nginx, WordPress and Tooling.
3. Create certificate from AWS certificate manager.
4. Create an External Application Load Balancer and Internal Application Load Balancer.
5. Create launch template for Bastion, Tooling, Nginx and WordPress.
6. Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress.
7. Create Elastic Filesystem.
8. Create Relational Database (RDS).

### Create Security Groups
  
  We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it.

- Create a file and name it `security.tf`, copy and paste the code below:

    ```tf
    # security group for alb, to allow acess from any where for HTTP and HTTPS traffic
    resource "aws_security_group" "ext-alb-sg" {
    name        = "ext-alb-sg"
    vpc_id      = aws_vpc.main.id
    description = "Allow TLS inbound traffic"

    ingress {
        description = "HTTP"
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    ingress {
        description = "HTTPS"
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = merge(
        var.tags,
        {
        Name = "ext-alb-sg"
        },
    )

    }


    # security group for bastion, to allow access into the bastion host from you IP
    resource "aws_security_group" "bastion_sg" {
    name        = "vpc_web_sg"
    vpc_id      = aws_vpc.main.id
    description = "Allow incoming HTTP connections."

    ingress {
        description = "SSH"
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = merge(
        var.tags,
        {
        Name = "Bastion-SG"
        },
    )
    }



    #security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance
    resource "aws_security_group" "nginx-sg" {
    name   = "nginx-sg"
    vpc_id = aws_vpc.main.id

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = merge(
        var.tags,
        {
        Name = "nginx-SG"
        },
    )
    }

    resource "aws_security_group_rule" "inbound-nginx-http" {
    type                     = "ingress"
    from_port                = 443
    to_port                  = 443
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.ext-alb-sg.id
    security_group_id        = aws_security_group.nginx-sg.id
    }

    resource "aws_security_group_rule" "inbound-bastion-ssh" {
    type                     = "ingress"
    from_port                = 22
    to_port                  = 22
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.bastion_sg.id
    security_group_id        = aws_security_group.nginx-sg.id
    }


    # security group for ialb, to have access only from nginx reverser proxy server
    resource "aws_security_group" "int-alb-sg" {
    name   = "my-alb-sg"
    vpc_id = aws_vpc.main.id

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = merge(
        var.tags,
        {
        Name = "int-alb-sg"
        },
    )

    }

    resource "aws_security_group_rule" "inbound-ialb-https" {
    type                     = "ingress"
    from_port                = 443
    to_port                  = 443
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.nginx-sg.id
    security_group_id        = aws_security_group.int-alb-sg.id
    }


    # security group for webservers, to have access only from the internal load balancer and bastion instance
    resource "aws_security_group" "webserver-sg" {
    name   = "my-asg-sg"
    vpc_id = aws_vpc.main.id

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = merge(
        var.tags,
        {
        Name = "webserver-sg"
        },
    )

    }

    resource "aws_security_group_rule" "inbound-web-https" {
    type                     = "ingress"
    from_port                = 443
    to_port                  = 443
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.int-alb-sg.id
    security_group_id        = aws_security_group.webserver-sg.id
    }

    resource "aws_security_group_rule" "inbound-web-ssh" {
    type                     = "ingress"
    from_port                = 22
    to_port                  = 22
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.bastion_sg.id
    security_group_id        = aws_security_group.webserver-sg.id
    }


    # security group for datalayer to alow traffic from websever on nfs and mysql port and bastion host on mysql port
    resource "aws_security_group" "datalayer-sg" {
    name   = "datalayer-sg"
    vpc_id = aws_vpc.main.id

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = merge(
        var.tags,
        {
        Name = "datalayer-sg"
        },
    )
    }

    resource "aws_security_group_rule" "inbound-nfs-port" {
    type                     = "ingress"
    from_port                = 2049
    to_port                  = 2049
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.webserver-sg.id
    security_group_id        = aws_security_group.datalayer-sg.id
    }

    resource "aws_security_group_rule" "inbound-mysql-bastion" {
    type                     = "ingress"
    from_port                = 3306
    to_port                  = 3306
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.bastion_sg.id
    security_group_id        = aws_security_group.datalayer-sg.id
    }

    resource "aws_security_group_rule" "inbound-mysql-webserver" {
    type                     = "ingress"
    from_port                = 3306
    to_port                  = 3306
    protocol                 = "tcp"
    source_security_group_id = aws_security_group.webserver-sg.id
    security_group_id        = aws_security_group.datalayer-sg.id
    }
    ```

    **Output:**
    ![SG](./images/security-groups.PNG)

### Create Certificate From Amazon Cerificate Manager

- Create `cert.tf` file and add the following code snippets to it.

  > NOTE: Read through to change the domain name to your own domain name including any other name that needs to be changed.

    ```tf
    # Create the certificate using a wildcard for all the domains created in oyindamola.gq
    resource "aws_acm_certificate" "opsmen" {
    domain_name       = "*.opsmen.online"
    validation_method = "DNS"
    key_algorithm     = "RSA_2048"

    lifecycle {
        create_before_destroy = true
    }
    }

    # calling the hosted zone
    data "aws_route53_zone" "opsmen" {
    name         = "opsmen.online"
    private_zone = false
    }

    # selecting validation method
    resource "aws_route53_record" "opsmen" {
    for_each = {
        for dvo in aws_acm_certificate.opsmen.domain_validation_options : dvo.domain_name => {
        name   = dvo.resource_record_name
        record = dvo.resource_record_value
        type   = dvo.resource_record_type
        }
    }

    allow_overwrite = true
    name            = each.value.name
    records         = [each.value.record]
    ttl             = 60
    type            = each.value.type
    zone_id         = data.aws_route53_zone.opsmen.zone_id
    }

    # validate the certificate through DNS method
    resource "aws_acm_certificate_validation" "opsmen" {
    certificate_arn         = aws_acm_certificate.opsmen.arn
    validation_record_fqdns = [for record in aws_route53_record.opsmen : record.fqdn]
    }

    # create records for tooling
    resource "aws_route53_record" "tooling" {
    zone_id = data.aws_route53_zone.opsmen.zone_id
    name    = "tooling.opsmen.online"
    type    = "A"

    alias {
        name                   = aws_lb.ext-alb.dns_name
        zone_id                = aws_lb.ext-alb.zone_id
        evaluate_target_health = true
    }
    }


    # create records for wordpress
    resource "aws_route53_record" "wordpress" {
    zone_id = data.aws_route53_zone.opsmen.zone_id
    name    = "wordpress.opsmen.online"
    type    = "A"

    alias {
        name                   = aws_lb.ext-alb.dns_name
        zone_id                = aws_lb.ext-alb.zone_id
        evaluate_target_health = true
    }
    }
    ```

    **Output 1:**
    ![ACM](./images/acm-01.PNG)

    **Output 2:**
    ![ACM](./images/acm-02.PNG)

### Create an External (Internet facing) Application Load Balancer (ALB)

- Create a file called `alb.tf`.

  First of all we will create the ALB, after which we create the `target group` and then, we create the `listener rule`.

    ```tf
    resource "aws_lb" "ext-alb" {
    name     = "ext-alb"
    internal = false
    security_groups = [
        aws_security_group.ext-alb-sg.id,
    ]

    subnets = [
        aws_subnet.public[0].id,
        aws_subnet.public[1].id
    ]

    tags = merge(
        var.tags,
        {
        Name = "ACS-ext-alb"
        },
    )

    ip_address_type    = "ipv4"
    load_balancer_type = "application"
    }
    ```

  To inform our ALB where to route the traffic we need to create a `target.tf` where we will have the configuration for the Target Groups to point to its targets:

    ```tf
    resource "aws_lb_target_group" "nginx-tgt" {
    health_check {
        interval            = 10
        path                = "/healthstatus"
        protocol            = "HTTPS"
        timeout             = 5
        healthy_threshold   = 5
        unhealthy_threshold = 2
    }
    name        = "nginx-tgt"
    port        = 443
    protocol    = "HTTPS"
    target_type = "instance"
    vpc_id      = aws_vpc.main.id
    }
    ```

  Then we will need to create a `listner.tf` file to contain configurations for this target groups:

    ```tf
    resource "aws_lb_listener" "nginx-listner" {
    load_balancer_arn = aws_lb.ext-alb.arn
    port              = 443
    protocol          = "HTTPS"
    certificate_arn   = aws_acm_certificate_validation.opsmen.certificate_arn

    default_action {
        type             = "forward"
        target_group_arn = aws_lb_target_group.nginx-tgt.arn
    }
    }
    ```

  Add the following outputs to `output.tf` to print them on screen:

    ```tf
    output "alb_dns_name" {
    value = aws_lb.ext-alb.dns_name
    }

    output "alb_target_group_arn" {
    value = aws_lb_target_group.nginx-tgt.arn
    }
    ```

### Create an Internal (Internal) Application Load Balancer (ALB)

For the Internal Load balancer we will follow the same concepts as with the external load balancer.

- Add the code snippets inside the `alb.tf` file

    ```tf
    # ----------------------------
    #Internal Load Balancers for webservers
    #---------------------------------

    resource "aws_lb" "ialb" {
    name     = "ialb"
    internal = true
    security_groups = [
        aws_security_group.int-alb-sg.id,
    ]

    subnets = [
        aws_subnet.private[0].id,
        aws_subnet.private[1].id
    ]

    tags = merge(
        var.tags,
        {
        Name = "ACS-int-alb"
        },
    )

    ip_address_type    = "ipv4"
    load_balancer_type = "application"
    }
    ```

  To inform our ALB where to route the traffic, we need to add the code snippets below inside the `target.tf` file.

    ```tf
    # --- target group  for wordpress -------

    resource "aws_lb_target_group" "wordpress-tgt" {
    health_check {
        interval            = 10
        path                = "/healthstatus"
        protocol            = "HTTPS"
        timeout             = 5
        healthy_threshold   = 5
        unhealthy_threshold = 2
    }

    name        = "wordpress-tgt"
    port        = 443
    protocol    = "HTTPS"
    target_type = "instance"
    vpc_id      = aws_vpc.main.id
    }


    # --- target group for tooling -------

    resource "aws_lb_target_group" "tooling-tgt" {
    health_check {
        interval            = 10
        path                = "/healthstatus"
        protocol            = "HTTPS"
        timeout             = 5
        healthy_threshold   = 5
        unhealthy_threshold = 2
    }

    name        = "tooling-tgt"
    port        = 443
    protocol    = "HTTPS"
    target_type = "instance"
    vpc_id      = aws_vpc.main.id
    }
    ```

  Then, we will need to add the listners for this target groups in the `listener.tf` file.

    ```tf
    resource "aws_lb_listener" "web-listener" {
    load_balancer_arn = aws_lb.ialb.arn
    port              = 443
    protocol          = "HTTPS"
    certificate_arn   = aws_acm_certificate_validation.opsmen.certificate_arn


    default_action {
        type             = "forward"
        target_group_arn = aws_lb_target_group.wordpress-tgt.arn
    }
    }

    # listener rule for tooling target

    resource "aws_lb_listener_rule" "tooling-listener" {
    listener_arn = aws_lb_listener.web-listener.arn
    priority     = 99

    action {
        type             = "forward"
        target_group_arn = aws_lb_target_group.tooling-tgt.arn
    }

    condition {
        host_header {
        values = ["tooling.opsmen.online"]
        }
    }
    }
    ```

    **Output 1:**
    ![TG](./images/target-groups.PNG)

    **Output 2:**
    ![ALB](./images/alb.PNG)

### Create an Auto Scaling Group (ASG)

  Now, we need to configure our ASG to be able to scale the EC2s in and out, depending on the application traffic.

  Before we start configuring an ASG, we need to create the launch template and the the AMI needed. For now we are going to use a random AMI from AWS, then in project 19, we will use `Packerto` create our ami.

  Based on the architecture we need Auto Scaling groups for bastion, nginx, wordpress and tooling, we will create two files; `asg-bastion-nginx.tf` will contain Launch Template and Austoscaling group for Bastion and Nginx, while `asg-wordpress-tooling.tf` will contain Launch Template and Austoscaling group for wordpress and tooling.

  - SNS-topic
  - SNS-notification
  - Austoscaling
  - Launch-template

- Create `asg-bastion-nginx.tf` and paste the code below:

    ```tf
    # creating sns topic for all the auto scaling groups
    resource "aws_sns_topic" "opsmen-sns" {
    name = "Default_CloudWatch_Alarms_Topic"
    }


    # creating notification for all the auto scaling groups
    resource "aws_autoscaling_notification" "opsmen_notifications" {
    group_names = [
        aws_autoscaling_group.bastion-asg.name,
        aws_autoscaling_group.nginx-asg.name,
        aws_autoscaling_group.wordpress-asg.name,
        aws_autoscaling_group.tooling-asg.name,
    ]
    notifications = [
        "autoscaling:EC2_INSTANCE_LAUNCH",
        "autoscaling:EC2_INSTANCE_TERMINATE",
        "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
        "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
    ]

    topic_arn = aws_sns_topic.opsmen-sns.arn
    }


    resource "random_shuffle" "az_list" {
    input = data.aws_availability_zones.available.names
    }


    # launch template for bastion

    resource "aws_launch_template" "bastion-launch-template" {
    image_id               = var.ami
    instance_type          = "t2.micro"
    vpc_security_group_ids = [aws_security_group.bastion_sg.id]

    iam_instance_profile {
        name = aws_iam_instance_profile.ip.id
    }

    key_name = var.keypair

    placement {
        availability_zone = "random_shuffle.az_list.result"
    }

    lifecycle {
        create_before_destroy = true
    }

    tag_specifications {
        resource_type = "instance"

        tags = merge(
        var.tags,
        {
            Name = "bastion-launch-template"
        },
        )
    }

    # create a file called bastion.sh and copy the bastion userdata from project 15 into it
    user_data = filebase64("${path.module}/bastion.sh")
    }

    # ---- Autoscaling for bastion  hosts


    resource "aws_autoscaling_group" "bastion-asg" {
    name                      = "bastion-asg"
    max_size                  = 2
    min_size                  = 1
    health_check_grace_period = 300
    health_check_type         = "ELB"
    desired_capacity          = 1

    vpc_zone_identifier = [
        aws_subnet.public[0].id,
        aws_subnet.public[1].id
    ]


    launch_template {
        id      = aws_launch_template.bastion-launch-template.id
        version = "$Latest"
    }
    tag {
        key                 = "Name"
        value               = "bastion-launch-template"
        propagate_at_launch = true
    }

    }


    # launch template for nginx

    resource "aws_launch_template" "nginx-launch-template" {
    image_id               = var.ami
    instance_type          = "t2.micro"
    vpc_security_group_ids = [aws_security_group.nginx-sg.id]

    iam_instance_profile {
        name = aws_iam_instance_profile.ip.id
    }

    key_name = var.keypair

    placement {
        availability_zone = "random_shuffle.az_list.result"
    }

    lifecycle {
        create_before_destroy = true
    }

    tag_specifications {
        resource_type = "instance"

        tags = merge(
        var.tags,
        {
            Name = "nginx-launch-template"
        },
        )
    }

    # create a file called nginx.sh and copy the nginx userdata from project 15 into it
    user_data = filebase64("${path.module}/nginx.sh")
    }


    # ------ Autoscslaling group for reverse proxy nginx ---------

    resource "aws_autoscaling_group" "nginx-asg" {
    name                      = "nginx-asg"
    max_size                  = 2
    min_size                  = 1
    health_check_grace_period = 300
    health_check_type         = "ELB"
    desired_capacity          = 1

    vpc_zone_identifier = [
        aws_subnet.public[0].id,
        aws_subnet.public[1].id
    ]

    launch_template {
        id      = aws_launch_template.nginx-launch-template.id
        version = "$Latest"
    }

    tag {
        key                 = "Name"
        value               = "nginx-launch-template"
        propagate_at_launch = true
    }


    }

    # attaching autoscaling group of nginx to external load balancer

    resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
    autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
    lb_target_group_arn    = aws_lb_target_group.nginx-tgt.arn
    }
    ```

- Create `asg-wordpress-tooling.tf` and paste the following code:

    ```tf
    # launch template for wordpress

    resource "aws_launch_template" "wordpress-launch-template" {
    image_id               = var.ami
    instance_type          = "t2.micro"
    vpc_security_group_ids = [aws_security_group.webserver-sg.id]

    iam_instance_profile {
        name = aws_iam_instance_profile.ip.id
    }

    key_name = var.keypair


    placement {
        availability_zone = "random_shuffle.az_list.result"
    }

    lifecycle {
        create_before_destroy = true
    }

    tag_specifications {
        resource_type = "instance"

        tags = merge(
        var.tags,
        {
            Name = "wordpress-launch-template"
        },
        )

    }

    # create a file called wordpress.sh and copy the wordpress userdata from project 15 into it.
    user_data = filebase64("${path.module}/wordpress.sh")
    }


    # ---- Autoscaling for wordpress application

    resource "aws_autoscaling_group" "wordpress-asg" {
    name                      = "wordpress-asg"
    max_size                  = 2
    min_size                  = 1
    health_check_grace_period = 300
    health_check_type         = "ELB"
    desired_capacity          = 1
    vpc_zone_identifier = [

        aws_subnet.private[0].id,
        aws_subnet.private[1].id
    ]


    launch_template {
        id      = aws_launch_template.wordpress-launch-template.id
        version = "$Latest"
    }
    tag {
        key                 = "Name"
        value               = "wordpress-asg"
        propagate_at_launch = true
    }
    }


    # attaching autoscaling group of wordpress application to internal loadbalancer
    resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
    autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
    lb_target_group_arn    = aws_lb_target_group.wordpress-tgt.arn
    }


    # launch template for tooling
    resource "aws_launch_template" "tooling-launch-template" {
    image_id               = var.ami
    instance_type          = "t2.micro"
    vpc_security_group_ids = [aws_security_group.webserver-sg.id]

    iam_instance_profile {
        name = aws_iam_instance_profile.ip.id
    }

    key_name = var.keypair


    placement {
        availability_zone = "random_shuffle.az_list.result"
    }

    lifecycle {
        create_before_destroy = true
    }

    tag_specifications {
        resource_type = "instance"

        tags = merge(
        var.tags,
        {
            Name = "tooling-launch-template"
        },
        )

    }

    # create a file called tooling.sh and copy the tooling userdata from project 15 into it
    user_data = filebase64("${path.module}/tooling.sh")
    }



    # ---- Autoscaling for tooling -----

    resource "aws_autoscaling_group" "tooling-asg" {
    name                      = "tooling-asg"
    max_size                  = 2
    min_size                  = 1
    health_check_grace_period = 300
    health_check_type         = "ELB"
    desired_capacity          = 1

    vpc_zone_identifier = [

        aws_subnet.private[0].id,
        aws_subnet.private[1].id
    ]

    launch_template {
        id      = aws_launch_template.tooling-launch-template.id
        version = "$Latest"
    }

    tag {
        key                 = "Name"
        value               = "tooling-launch-template"
        propagate_at_launch = true
    }
    }

    # attaching autoscaling group of  tooling application to internal loadbalancer
    resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
    autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
    lb_target_group_arn    = aws_lb_target_group.tooling-tgt.arn
    }
    ```

  **Output 1:**
  ![LT](./images/launch-templates.PNG)

  **Output 2:**
  ![ASG](./images/ASG.PNG)

  **Output 3:**
  ![EC2](./images/instances.PNG)

## Storage and Database

- Create Elastic File System (EFS)

  In order to create an EFS you need to create a `KMS key`.

  AWS Key Management Service (KMS) makes it easy for you to create and manage cryptographic keys and control their use across a wide range of AWS services and also in your applications.

  Add the following code to `kms.tf` file:

    ```tf
    # create key from key management system
    resource "aws_kms_key" "opsmen-kms" {
    description = "KMS key "
    policy      = <<EOF
    {
    "Version": "2012-10-17",
    "Id": "kms-key-policy",
    "Statement": [
        {
        "Sid": "Enable IAM User Permissions",
        "Effect": "Allow",
        "Principal": { "AWS": "arn:aws:iam::${var.account_no}:user/terraform" },
        "Action": "kms:*",
        "Resource": "*"
        }
    ]
    }
    EOF
    }

    # create key alias
    resource "aws_kms_alias" "alias" {
    name          = "alias/kms"
    target_key_id = aws_kms_key.opsmen-kms.key_id
    }
    ```

  Let us create EFS and it's mount targets. Add the following code to `efs.tf` file:

    ```tf
    # create Elastic file system
    resource "aws_efs_file_system" "opsmen-efs" {
    encrypted  = true
    kms_key_id = aws_kms_key.opsmen-kms.arn

    tags = merge(
        var.tags,
        {
        Name = "opsmen-efs"
        },
    )
    }

    # set first mount target for the EFS
    resource "aws_efs_mount_target" "subnet-1" {
    file_system_id  = aws_efs_file_system.opsmen-efs.id
    subnet_id       = aws_subnet.private[2].id
    security_groups = [aws_security_group.datalayer-sg.id]
    }


    # set second mount target for the EFS
    resource "aws_efs_mount_target" "subnet-2" {
    file_system_id  = aws_efs_file_system.opsmen-efs.id
    subnet_id       = aws_subnet.private[3].id
    security_groups = [aws_security_group.datalayer-sg.id]
    }

    # create access point for wordpress
    resource "aws_efs_access_point" "wordpress" {
    file_system_id = aws_efs_file_system.opsmen-efs.id

    posix_user {
        gid = 0
        uid = 0
    }

    root_directory {
        path = "/wordpress"

        creation_info {
        owner_gid   = 0
        owner_uid   = 0
        permissions = 0755
        }

    }

    }

    # create access point for tooling
    resource "aws_efs_access_point" "tooling" {
    file_system_id = aws_efs_file_system.opsmen-efs.id
    posix_user {
        gid = 0
        uid = 0
    }

    root_directory {

        path = "/tooling"

        creation_info {
        owner_gid   = 0
        owner_uid   = 0
        permissions = 0755
        }

    }
    }
    ```

    **Output 1:**
    ![EFS](./images/efs-01.PNG)

    **Output 2:**
    ![EFS](./images/efs-02.PNG)

### Create MySQL RDS

  Let us create the RDS itself using this snippet of code in the `rds.tf` file:

   ```tf
    # This section will create the subnet group for the RDS instance using the private subnet
    resource "aws_db_subnet_group" "opsmen-db-subnet-group" {
    name       = "opsmen-db-subnet-group"
    subnet_ids = [aws_subnet.private[2].id, aws_subnet.private[3].id]

    tags = merge(
        var.tags,
        {
        Name = "opsmen-rds"
        },
    )
    }

    # create the RDS instance with the subnets group
    resource "aws_db_instance" "opsmen-rds" {
    allocated_storage      = 50
    storage_type           = "gp3"
    engine                 = "mysql"
    engine_version         = "8.0.35"
    instance_class         = "db.t3.micro"
    db_name                = "opsmendb"
    identifier             = "opsmen-database"
    username               = var.master-username
    password               = var.master-password
    parameter_group_name   = "default.mysql8.0"
    db_subnet_group_name   = aws_db_subnet_group.opsmen-db-subnet-group.name
    skip_final_snapshot    = true
    vpc_security_group_ids = [aws_security_group.datalayer-sg.id]
    multi_az               = "true"
    }
   ```

  **Output 1:**
  ![RDS](./images/rds-01.PNG)

  **Output 2:**
  ![RDS](./images/rds-02.PNG)

  Before applying, please note that we gave reference to some variables in our resources that have not been declared in the `variables.tf` file. Go through the entire code and spot these variables and declare them in the `variables.tf` file.

  If you have done that well, your file should look like this one below:

   ```tf
    variable "region" {
    type        = string
    description = "The region to deploy resources"
    }

    variable "vpc_cidr" {
    type        = string
    description = "The VPC cidr"
    }

    variable "enable_dns_support" {
    type = bool
    }

    variable "enable_dns_hostnames" {
    type = bool
    }

    variable "preferred_number_of_public_subnets" {
    type        = number
    description = "Number of public subnets"
    }

    variable "preferred_number_of_private_subnets" {
    type        = number
    description = "Number of private subnets"
    }

    variable "name" {
    type    = string
    default = "ACS"

    }

    variable "tags" {
    description = "A mapping of tags to assign to all resources."
    type        = map(string)
    default     = {}
    }


    variable "ami" {
    type        = string
    description = "AMI ID for the launch template"
    }


    variable "keypair" {
    type        = string
    description = "key pair for the instances"
    }

    variable "account_no" {
    type        = number
    description = "the account number"
    }


    variable "master-username" {
    type        = string
    description = "RDS admin username"
    }

    variable "master-password" {
    type        = string
    description = "RDS master password"
    }
   ```

  We are almost done but we need to update the last file which is `terraform.tfvars` file. In this file we are going to declare the values for the variables in our `varibales.tf` file.

  Open the `terraform.tfvars` file and add the code below:

    ```tf
    region = "us-east-1"

    name = "ops_men"

    vpc_cidr = "172.16.0.0/16"

    enable_dns_support = true

    enable_dns_hostnames = true

    preferred_number_of_public_subnets = 2

    preferred_number_of_private_subnets = 4

    ami = "ami-0b0af3577fe5e3532"

    keypair = "opsmenkey"

    tags = {
    Environment     = "production"
    Owner-Email     = "oladiposamson34@gmail.com"
    Managed-By      = "Terraform"
    Billing-Account = "1234567890"
    }

    account_no = "1234567890"


    master-username = "samson"


    master-password = "makeitrain"
    ```

  At this point, you shall have pretty much all the infrastructure elements ready to be deployed automatically. Try to `plan` and `apply` your Terraform codes, explore the resources in AWS console and make sure you `destroy` them right away to avoid massive costs.

  **Output 1:**
  ![Plan](./images/terraform-plan.PNG)

  **Output 2:**
  ![Apply](./images/terraform-apply.PNG)

  **Output 3:**
  ![Destroy](./images/terraform-destroy.PNG)

---

### Congratulations

Now you have fully automated creation of AWS Infrastructure for 2 websites with Terraform. In the next project, we will further enhance our codes by refactoring and introducing more exciting Terraform concepts! Go ahead and continue your PBL journey with us!

---

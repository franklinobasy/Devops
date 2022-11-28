# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![](images/tooling_project_15.png)

## Starting Off Your AWS Cloud Project
There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit [Watch How To Do This Here](https://youtu.be/9PQYCc_20-Q)

- Create an AWS Master account. (Also known as Root Account)
- Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
- Move the DevOps account into the Dev OU.
- Login to the newly created AWS account using the new email address.

1. Create a free domain name for your fictitious company at Freenom domain registrar here.

2. Create a hosted zone in AWS, and map it to your free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A)

![](images/1-a-14.png)

NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

- Project: <Give your project a name>
- Environment: <dev>
- Automated: <No> (If you create a recource using an automation tool, it would be <Yes>)

## Setting Up Infrastucture
1. Create a VPC

![](images/2-a-14.png)

2. Create subnets as shown in the architecture
   
   ![](images/subnets.png)

3. Create a route table and associate it with public subnets
   
   ![](images/route-public-1a.png)

   ![](images/route-public-1b.png)

   ![](images/route-public-1c.png)

   ![](images/route-public-1d.png)

4. Create a route table and associate it with private subnets
   
   ![](images/route-private-1a.png)

5. Create an Internet Gateway
   
   ![](images/igw-1a.png)

   ![](images/igw-1b.png)

   ![](images/igw-1c.png)

   ![](images/igw-1d.png)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
   
   ![](images/edit-public-route-1a.png)

   ![](images/edit-public-route-1b.png)

   ![](images/edit-public-route-1c.png)

7. Create 3 Elastic IPs
   
   ![](images/elastic-IP-1a.png)

   ![](images/elastic-IP-1b.png)

   ![](images/elastic-IP-1c.png)

8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
   
   ![](images/NAT-1a.png)

   ![](images/NAT-1b.png)

   ![](images/NAT-1c.png)

9. Create a Security Group for:

- Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type `curl www.canhazip.com`
Application Load Balancer: ALB will be available from the Internet

![](images/sgr-bastion.png)

- Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

## Proceed With Compute Resources
You will need to set up and configure compute resources inside your VPC. The recources related to compute are:

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)

### TLS Certficate

![](images/cert-1a.png)

![](images/cert-1b.png)

![](images/cert-1c.png)

![](images/cert-1d.png)

![](images/cert-1e.png)


### Amazon Elastic File System

![](images/efs-1a.png)

![](images/efs-1b.png)

![](images/efs-1c.png)

![](images/efs-1d.png)

![](images/efs-1e.png)

![](images/efs-1f.png)

![](images/efs-1g.png)

![](images/efs-1h.png)

- Create Access points for `wordpress` and `tooling`
  
![](images/efs-1i1.png)
![](images/efs-1i2.png)
![](images/efs-1i3.png)

![](images/efs-1j.png)

### Amazon RDS - Relational Database Services
- Create Key Management Service
  
  ![](images/kms-1a.png)

  ![](images/kms-1b.png)

  ![](images/kms-1c.png)

  ![](images/kms-1d.png)

  ![](images/kms-1e.png)

- Create Subnet groups
  
  ![](images/db-subnet-1a.png)

  ![](images/db-subnet-1bi.png)
  ![](images/db-subnet-1bii.png)

  ![](images/db-subnet-1c.png)

- Create Database
  
  ![](images/rds-1a.png)

  ![](images/rds-1bi.png)

  ![](images/rds-1bii.png)

  ![](images/rds-1biii.png)

  ![](images/rds-1biv.png)

  ![](images/rds-1bv.png)
  

## Set Up Compute Resources for Nginx

### Creat AMIs for Launch Templates
- Created 3 EC2 Instances
  
  ![](images/AMI-1a.png)

  ![](images/AMI-1b.png)

  1. connect to Bastion instance via ssh
     Install the following components

     ```
     sudo su -
     sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
     sudo dnf install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
     yum install wget vim python3 telnet htop git mysql net-tools chrony -y
     systemctl start chronyd
     systemctl enable chronyd
     ```

   2. connect to Nginx intance via ssh
    Install the following components
     
    ```
    sudo su -
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo dnf install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd

    # configure selinux policies for nginx
    setsebool -P httpd_can_network_connect=1
    setsebool -P httpd_can_network_connect_db=1
    setsebool -P httpd_execmem=1
    setsebool -P httpd_use_nfs 1

    # install amazon efs utils for mounting the target on the Elastic file system
    git clone https://github.com/aws/efs-utils
    cd efs-utils
    yum install -y make
    yum install -y rpm-build
    make rpm 
    yum install -y  ./build/amazon-efs-utils*rpm

    # seting up self-signed certificate for the nginx instance
    sudo mkdir /etc/ssl/private
    sudo chmod 700 /etc/ssl/private
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt
    sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

    ```

  3. connect to the webserver instance
     
    ```
    sudo su -
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo dnf install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd

    # configure selinux policies for nginx
    setsebool -P httpd_can_network_connect=1
    setsebool -P httpd_can_network_connect_db=1
    setsebool -P httpd_execmem=1
    setsebool -P httpd_use_nfs 1

    # install amazon efs utils for mounting the target on the Elastic file system
    git clone https://github.com/aws/efs-utils
    cd efs-utils
    yum install -y make
    yum install -y rpm-build
    make rpm 
    yum install -y  ./build/amazon-efs-utils*rpm

    ## seting up self-signed certificate for the apache  webserver instance
    yum install -y mod_ssl
    openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt
    vi /etc/httpd/conf.d/ssl.conf
    ```

- Create AMI for each of the Instances
  
  ![](images/create-ami-1a.png)

  ![](images/create-ami-1b.png)

  ![](images/create-ami-1c.png)

- Create Target Groups
  Create target groups for `nginx`, `wordpress` and `tooling`

  ![](images/tg-1a.png)

  ![](images/tg-1bi.png)

  ![](images/tg-1bii.png)

  ![](images/tg-1biii.png)


### Create Load Balancer

Application Load Balancer To Route Traffic To NGINX
Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.


![](images/lb-1a.png)


### Create an Internet facing ALB

- Ensure that it listens on HTTPS protocol (TCP port 443)

- Ensure the ALB is created within the appropriate VPC | AZ | Subnets

- Choose the Certificate from ACM

- Select Security Group

- Select Nginx Instances as the target group

- Application Load Balancer To Route Traffic To Web Servers

- Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

- To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

### Create an Internal ALB

- Ensure that it listens on HTTPS protocol (TCP port 443)

- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM

- Select Security Group

- Select webserver Instances as the target group

- Ensure that health check passes for the target group

![](./images/loadbalancers.png)

**NOTE:** This process must be repeated for both WordPress and Tooling websites.

- Route traffic coming from the nginx server into the internal loadbalancer by sending traffic to the respective target group based on the url being requested by the user.
  
![](./images/adding%20rules.png)


## Creating Databases for Wordpress and Tooling Sites on MySQL rds

- Login into the MySQL RDS from the bastion server

- Create databases 

![](./images/creating%20database%20tooling%20and%20wordpress.PNG)  


![](./images/tooling%20webpage.PNG)
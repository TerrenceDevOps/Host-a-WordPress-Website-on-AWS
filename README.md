# Hosting a WordPress Website on AWS

This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

## Architecture Overview

The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

- A Virtual Private Cloud (VPC) with public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
- An Internet Gateway to allow communication between instances in the VPC and the internet.
- Security Groups acting as a virtual firewall to control inbound and outbound traffic.
- Public Subnets used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
- Private Subnets for web servers to enhance security.
- EC2 Instance Connect Endpoint for secure SSH access.
- An Application Load Balancer with a target group to distribute incoming web traffic across multiple EC2 instances.
- An Auto Scaling Group to automatically adjust the number of EC2 instances based on traffic, ensuring scalability and resilience.
- Amazon RDS for a managed relational database service.
- Amazon EFS for a scalable, elastic file storage system.
- AWS Certificate Manager for managing SSL/TLS certificates.
- AWS Simple Notification Service (SNS) for notifications related to the Auto Scaling Group activities.
- Amazon Route 53 for domain name registration and DNS management.

## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

```bash
# Switch to root user
sudo su

# Update the software packages on the EC2 instance
sudo yum update -y

# Create an HTML directory
sudo mkdir -p /var/www/html

# Environment variable for EFS
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server, enable it to start on boot, and start the server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 along with necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# Download and set up WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create and configure wp-config.php
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Restart Apache
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

This script ensures new instances are correctly configured within the Auto Scaling Group.

```bash
#!/bin/bash
sudo yum update -y

# Install Apache
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server

# Start and enable MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Environment variable for EFS
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
```

## Lessons Learned

While setting up this project, I encountered challenges with configuring the DNS using Amazon Route 53. Initially, I struggled with proper domain name resolution and routing traffic correctly. However, through research and troubleshooting, I learned:

- The importance of correctly configuring hosted zones and record sets in Route 53.
- The role of TTL (Time to Live) in DNS propagation and how it affects changes.
- How to use dig and nslookup commands to debug DNS-related issues.
- The need to properly link the Application Load Balancer’s DNS name to the domain using an Alias record.

Overcoming these obstacles deepened my understanding of DNS management and enhanced my overall DevOps skills.

## How to Use

1. Clone this repository to your local machine.
2. Follow the AWS documentation to create the required resources (VPC, subnets, Internet Gateway, etc.).
3. Use the provided scripts to set up the WordPress application on EC2 instances.
4. Configure the Auto Scaling Group, Load Balancer, and other services as per the architecture.
5. Access the WordPress website through the Load Balancer's DNS name.

## Contributing

If you find any issues or have suggestions for improvements, feel free to submit a pull request or open an issue in this repository.


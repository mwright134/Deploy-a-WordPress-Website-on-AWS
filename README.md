---

# WordPress Website Deployment on AWS

Welcome to the README file for hosting a WordPress website on Amazon Web Services (AWS)! This project uses AWS resources and deployment scripts to facilitate the setup and management of a WordPress site.


###Project Overview


## Architecture Overview

The project contains the following key components and steps:

   1.) **Virtual Private Cloud (VPC)**:
        Configured a VPC with public and private subnets spanning two availability zones for improved reliability and fault tolerance.

   2.) **Internet Gateway**:
        Deployed an Internet Gateway to enable connectivity between VPC instances and the internet.

   3.) **Security Groups**:
        Established Security Groups to act as a firewall to control network traffic.

   4.) **Availability Zones**:
        Utilized two Availability Zones to enhance system resilience.

   5.) **Public Subnets**:
        Used Public Subnets for components like the NAT Gateway and Application Load Balancer.

   6.) **Private Subnets**:
	Used for web servers to enhance security.

   7.) **EC2 Instance Connect Endpoint**:
        Implemented EC2 Instance Connect Endpoint for secure connections to assets within both public and private subnets.

   8.) **Web Servers (EC2 instances)**:
        Positioned web servers (EC2 instances) within Private Subnets for heightened security.

   9.)** NAT Gateway**:
        Enabled instances in private subnets to access the Internet via the NAT Gateway.

   10.) **WordPress Hosting**:
        Hosted the website on EC2 Instances.

   11.) **Load Balancing**:
        Utilized an Application Load Balancer and a target group to evenly distribute web traffic to an Auto Scaling Group of EC2 instances across multiple Availability Zones.

   12.) **Auto Scaling Group**:
        Utilized an Auto Scaling Group to automatically manage EC2 instances based on traffic, ensuring availability, scalability, fault tolerance, and elasticity.

   13.) **Version Control**:
        Stored web files on GitHub for version control and collaboration.

   14.) **SSL/TLS Encryption**:
        Secured application communications using AWS Certificate Manager.

   15.) **Monitoring**:
        Configured Simple Notification Service (SNS) to alert about activities within the Auto Scaling Group.

   16.) **Domain Name System (DNS)**:
        Registered the domain name and set up DNS records using Route 53.

   17.) **Amazon EFS**:
        Utilized Amazon EFS for a scalable, eleastic file storage system.

   18.) **Amazon RDS**:
        Used Amazon RDS for the managed relational database.


## Deployment Scripts

### WordPress Installation Script

This script installs WordPress on an EC2 instance. It performs the following actions:

    -Updates the software packages on the EC2 instance.
    -Creates an HTML directory and mounts Amazon EFS to store web files.
    -Installs Apache web server, PHP, MySQL server, and necessary PHP extensions for WordPress.
    -Downloads WordPress files, extracts them, configures wp-config.php, and restarts the webserver.

```bash
#!/bin/bash

# Update the software packages on the EC2 instance 
sudo yum update -y

# Create an HTML directory 
sudo mkdir -p /var/www/html

# Environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 along with several necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# Download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# Restart the webserver
sudo service httpd restart


### Auto Scaling Group Launch Template Script (auto_scaling_setup.sh)

This script sets up an Auto Scaling Group launch template. It performs the following actions:

    Updates the software packages on the EC2 instance.
    Installs Apache web server, PHP, MySQL server, and necessary PHP extensions for WordPress.
    Mounts Amazon EFS to store web files, sets permissions, and restarts the webserver.

#!/bin/bash

# Update the software packages on the EC2 instance 
sudo yum update -y

# Install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 along with several necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Environment variable
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart the webserver
sudo service httpd restart


## Getting Started

To deploy the WordPress website on AWS, follow these steps:

    Clone the GitHub repository containing the deployment scripts and configuration files.
    Ensure you have an AWS account with appropriate permissions.
    Modify the scripts as per your requirements, replacing placeholders like EFS_DNS_NAME with actual values specific to your AWS environment.
    Run the scripts on the respective EC2 instances as described in the repository.

Note: Remember to replace placeholders like "EFS_DNS_NAME" with the EFS Name specific to your AWS environment.

## Contributions

If you have any recommendations or contributions please do not hesiate to fork the repository and submit a pull request with your enhancements!

## Support

If you encounter any issues or have questions, refer to the AWS documentation or seek assistance from AWS support.

Remember, 

Happy Deploying! 🚀

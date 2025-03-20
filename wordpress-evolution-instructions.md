# WordPress Architecture Evolution - Step by Step Instructions

This guide walks through the evolution of a WordPress architecture from a single server setup to a fully elastic infrastructure on AWS. Perfect for AWS SAA-03 exam preparation to understand real-world architectures.

## Overview of the Project

This project demonstrates how to evolve a WordPress application:
1. **Stage 1**: Single EC2 instance (manual setup)
2. **Later Stages**: Evolution to a scalable, elastic architecture

## Prerequisites

- AWS account with admin access
- AWS region set to us-east-1 (N. Virginia)
- Basic understanding of AWS services (EC2, VPC, RDS)

## Stage 1: Single Server WordPress Setup

### Step 1: Deploy the VPC CloudFormation Template

1. First, deploy the VPC CloudFormation template which contains the networking infrastructure
2. Wait for the stack to reach CREATE_COMPLETE status before proceeding

### Step 2: Create EC2 Instance

1. Go to EC2 console: https://console.aws.amazon.com/ec2/v2/home?region=us-east-1
2. Click "Launch Instance"
3. For name, enter "Wordpress-Manual"
4. Select Amazon Linux 2023 AMI (64-bit x86)
5. Choose the free tier eligible instance type (t2 or t3.micro)
6. Under Key Pair, select "Proceed without a KeyPair"
7. Network Settings:
   - VPC: A4LVPC
   - Subnet: sn-Pub-A
   - Auto-assign public IP: Enable
   - Auto-assign IPv6 IP: Enable
8. Security Group: Select existing "A4LVPC-SGWordpress"
9. Advanced Details:
   - IAM Instance Profile: A4LVPC-WordpressInstanceProfile (IMPORTANT!)
   - Credit Specification: Standard
10. Click "Launch Instance"

### Step 3: Create SSM Parameters for WordPress

Create the following parameters in SSM Parameter Store:

| Name | Type | Value |
|------|------|-------|
| /A4L/Wordpress/DBUser | String | a4lwordpressuser |
| /A4L/Wordpress/DBName | String | a4lwordpressdb |
| /A4L/Wordpress/DBEndpoint | String | localhost |
| /A4L/Wordpress/DBPassword | SecureString | 4n1m4l54L1f3 |
| /A4L/Wordpress/DBRootPassword | SecureString | 4n1m4l54L1f3 |

### Step 4: Connect to the Instance and Install WordPress

1. Connect to the instance using Session Manager
2. Switch to root and fetch the SSM parameters:
   ```bash
   sudo bash
   cd
   
   # Fetch parameters from SSM (if disconnected, rerun this block)
   DBPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBPassword --with-decryption --query Parameters[0].Value)
   DBPassword=`echo $DBPassword | sed -e 's/^"//' -e 's/"$//'`

   DBRootPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBRootPassword --with-decryption --query Parameters[0].Value)
   DBRootPassword=`echo $DBRootPassword | sed -e 's/^"//' -e 's/"$//'`

   DBUser=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBUser --query Parameters[0].Value)
   DBUser=`echo $DBUser | sed -e 's/^"//' -e 's/"$//'`

   DBName=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBName --query Parameters[0].Value)
   DBName=`echo $DBName | sed -e 's/^"//' -e 's/"$//'`

   DBEndpoint=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBEndpoint --query Parameters[0].Value)
   DBEndpoint=`echo $DBEndpoint | sed -e 's/^"//' -e 's/"$//'`
   ```

3. Install updates:
   ```bash
   sudo dnf -y update
   ```

4. Install prerequisites and web server:
   ```bash
   sudo dnf install wget php-mysqlnd httpd php-fpm php-mysqli mariadb105-server php-json php php-devel stress -y
   ```

5. Enable and start services:
   ```bash
   sudo systemctl enable httpd
   sudo systemctl enable mariadb
   sudo systemctl start httpd
   sudo systemctl start mariadb
   ```

6. Set MariaDB root password:
   ```bash
   sudo mysqladmin -u root password $DBRootPassword
   ```

7. Download and extract WordPress:
   ```bash
   sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
   cd /var/www/html
   sudo tar -zxvf latest.tar.gz
   sudo cp -rvf wordpress/* .
   sudo rm -R wordpress
   sudo rm latest.tar.gz
   ```

8. Configure WordPress:
   ```bash
   sudo cp ./wp-config-sample.php ./wp-config.php
   sudo sed -i "s/'database_name_here'/'$DBName'/g" wp-config.php
   sudo sed -i "s/'username_here'/'$DBUser'/g" wp-config.php
   sudo sed -i "s/'password_here'/'$DBPassword'/g" wp-config.php
   ```

9. Fix filesystem permissions:
   ```bash
   sudo usermod -a -G apache ec2-user   
   sudo chown -R ec2-user:apache /var/www
   sudo chmod 2775 /var/www
   sudo find /var/www -type d -exec chmod 2775 {} \;
   sudo find /var/www -type f -exec chmod 0664 {} \;
   ```

10. Create database and user:
    ```bash
    sudo echo "CREATE DATABASE $DBName;" >> /tmp/db.setup
    sudo echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" >> /tmp/db.setup
    sudo echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup
    sudo echo "FLUSH PRIVILEGES;" >> /tmp/db.setup
    sudo mysql -u root --password=$DBRootPassword < /tmp/db.setup
    sudo rm /tmp/db.setup
    ```

### Step 5: Configure WordPress

1. Open the WordPress site using the EC2 instance's public IP
2. Complete the setup:
   - Site Title: Catagram
   - Username: admin
   - Password: 4n1m4l54L1f3
   - Email: Your email address
3. Log in and create a sample post

### Step 6: Test the Limitations of Single-Server Architecture

1. Stop and start the EC2 instance
2. Notice how the public IP address changes
3. Observe how this breaks the WordPress site (images point to old IP)

## Limitations of This Architecture

1. **Manual Installation**: Time-consuming and error-prone
2. **Co-located Database**: DB and app on same instance, neither can scale independently
3. **Local Media Storage**: Images and files stored locally on the instance
4. **Direct Connection**: No load balancing or health checks
5. **Hardcoded IP**: Database contains the instance's IP, which changes on restart

## Next Steps

In later stages, you'll evolve this architecture to address these limitations by implementing:
- Automated deployment
- Separate database tier (RDS)
- Shared storage (EFS)
- Load balancing (ALB)
- Auto Scaling Group for WordPress
- Content distribution (CloudFront)

## Exam Tips

- The single-server architecture demonstrates the limitations that more advanced architectures solve
- SSM Parameter Store provides a way to securely store configuration data
- Understand the impact of instance restarts on public IP addresses
- Recognize the need for persistent storage separated from compute instances
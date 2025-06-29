# AWS & Linux Project Solutions

This document contains detailed solutions for the projects listed in `project.md`. Each solution includes step-by-step instructions and relevant code snippets.

## Table of Contents
- [1. Linux Server Setup & Basic Administration](#1-linux-server-setup--basic-administration)
- [2. Static Website Hosting on S3](#2-static-website-hosting-on-s3)
- [3. LAMP/LEMP Stack Deployment](#3-lamp-lemp-stack-deployment)
- [4. Automated Backup System](#4-automated-backup-system)
- [5. CI/CD Pipeline with AWS CodePipeline](#5-cicd-pipeline-with-aws-codepipeline)
- [6. Docker Containerization](#6-docker-containerization)
- [7. Infrastructure as Code with Terraform](#7-infrastructure-as-code-with-terraform)
- [8. Monitoring and Logging Solution](#8-monitoring-and-logging-solution)
- [9. Serverless Application](#9-serverless-application)
- [10. Multi-Tier Architecture](#10-multi-tier-architecture)
- [11. Kubernetes Cluster on AWS](#11-kubernetes-cluster-on-aws)
- [12. Security Hardening and Compliance](#12-security-hardening-and-compliance)
- [13. Bash Scripting Challenges](#13-bash-scripting-challenges)
- [14. Linux Server Hardening](#14-linux-server-hardening)
- [15. Build a Custom Linux Distribution](#15-build-a-custom-linux-distribution)

## 1. Linux Server Setup & Basic Administration

### Steps:
1. **Launch an EC2 Instance**
   - Log in to AWS Console
   - Navigate to EC2 > Instances > Launch Instance
   - Select Amazon Linux 2 AMI
   - Choose t2.micro (free tier eligible)
   - Configure instance details (defaults are fine for learning)
   - Add storage (8GB is sufficient)
   - Add tags (e.g., Name: Linux-Server)
   - Configure security group (allow SSH on port 22 from your IP)
   - Create a new key pair and download the .pem file

2. **Connect to the Instance**
   ```bash
   chmod 400 your-key.pem
   ssh -i "your-key.pem" ec2-user@your-instance-public-dns
   ```

3. **Basic Administration Tasks**
   - Update packages:
     ```bash
     sudo yum update -y
     ```
   - Create a new user:
     ```bash
     sudo adduser newuser
     sudo passwd newuser
     sudo usermod -aG wheel newuser  # Add to sudo group
     ```
   - Basic file operations:
     ```bash
     mkdir testdir
     touch testfile.txt
     chmod 755 testfile.txt
     chown newuser:newgroup testfile.txt
     ```

## 2. Static Website Hosting on S3

### Steps:
1. **Create an S3 Bucket**
   - Go to S3 in AWS Console
   - Click "Create bucket"
   - Enter a unique bucket name (e.g., yourname-website)
   - Uncheck "Block all public access" and acknowledge
   - Enable static website hosting in bucket properties
   - Set index.html as the index document

2. **Upload Website Files**
   - Create a simple index.html:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>My Static Website</title>
     </head>
     <body>
         <h1>Welcome to My Website</h1>
         <p>This is hosted on AWS S3!</p>
     </body>
     </html>
     ```
   - Upload to your S3 bucket
   - Set the bucket policy to make objects public:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "PublicReadGetObject",
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::your-bucket-name/*"
             }
         ]
     }
     ```

3. **Access Your Website**
   - The website will be available at: `http://your-bucket-name.s3-website-region.amazonaws.com`

## 3. LAMP/LEMP Stack Deployment

### LAMP Stack Setup (Ubuntu):

1. **Update System**
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

2. **Install Apache**
   ```bash
   sudo apt install apache2 -y
   sudo systemctl start apache2
   sudo systemctl enable apache2
   ```

3. **Install MySQL**
   ```bash
   sudo apt install mysql-server -y
   sudo mysql_secure_installation
   ```

4. **Install PHP**
   ```bash
   sudo apt install php libapache2-mod-php php-mysql -y
   ```

5. **Test PHP**
   ```bash
   echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
   ```
   Visit: `http://your-server-ip/info.php`

## 4. Automated Backup System

### Bash Script for EC2 Backup:
```bash
#!/bin/bash

# Variables
INSTANCE_ID="i-1234567890abcdef0"  # Replace with your instance ID
VOLUME_ID="vol-0123456789abcdef0"  # Replace with your volume ID
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Create snapshot
SNAPSHOT_ID=$(aws ec2 create-snapshot \
    --volume-id $VOLUME_ID \
    --description "Automatic backup $(date +%Y-%m-%d)" \
    --query 'SnapshotId' --output text)

echo "Created snapshot: $SNAPSHOT_ID"

# Optional: Copy backup to S3
aws s3 cp $BACKUP_DIR s3://your-backup-bucket/ec2-backups/ --recursive

# Clean up old snapshots (older than 30 days)
aws ec2 describe-snapshots --filters "Name=description,Values=Automatic backup*" \
    --query "Snapshots[?(StartTime<='$(date --date='30 days ago' +%Y-%m-%d)')].SnapshotId" \
    --output text | xargs -r -t -n 1 aws ec2 delete-snapshot --snapshot-id

# Add to crontab (runs daily at 2 AM)
# 0 2 * * * /path/to/backup_script.sh
```

## 5. CI/CD Pipeline with AWS CodePipeline

### Steps:
1. **Set up CodeCommit Repository**
   ```bash
   # Install Git
   sudo yum install git -y
   
   # Configure Git
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   
   # Clone your CodeCommit repo
   git clone https://git-codecommit.region.amazonaws.com/v1/repos/YourRepoName
   ```

2. **Create buildspec.yml**
   ```yaml
   version: 0.2
   
   phases:
     install:
       runtime-versions:
         nodejs: 14
       commands:
         - echo Installing dependencies...
         - npm install
     build:
       commands:
         - echo Build started on `date`
         - npm run build
     post_build:
       commands:
         - echo Build completed on `date`
   artifacts:
     files:
       - '**/*'
     base-directory: 'build'
   ```

3. **Set up Pipeline in AWS Console**
   - Go to CodePipeline
   - Create new pipeline
   - Add source (CodeCommit)
   - Add build (CodeBuild)
   - Add deploy (S3/ECS/EB)

## 6. Docker Containerization

### Simple Node.js App with Docker

1. **Create package.json**
   ```json
   {
     "name": "node-docker-app",
     "version": "1.0.0",
     "description": "Node.js on Docker",
     "main": "server.js",
     "scripts": {
       "start": "node server.js"
     },
     "dependencies": {
       "express": "^4.17.1"
     }
   }
   ```

2. **Create server.js**
   ```javascript
   const express = require('express');
   const app = express();
   const PORT = 3000;
   
   app.get('/', (req, res) => {
     res.send('Hello from Docker!');
   });
   
   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

3. **Create Dockerfile**
   ```dockerfile
   FROM node:14
   WORKDIR /usr/src/app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["node", "server.js"]
   ```

4. **Build and Run**
   ```bash
   # Build the image
   docker build -t node-app .
   
   # Run the container
   docker run -p 3000:3000 -d node-app
   ```

   Visit: `http://localhost:3000`

## 7. Infrastructure as Code with Terraform

### Basic EC2 Instance with Terraform

1. **Create main.tf**
   ```hcl
   provider "aws" {
     region = "us-west-2"
   }
   
   resource "aws_instance" "example" {
     ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI
     instance_type = "t2.micro"
     
     tags = {
       Name = "Terraform-Example"
     }
   }
   ```

2. **Initialize and Apply**
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

## 8. Monitoring and Logging Solution

### CloudWatch Agent Setup

1. **Install CloudWatch Agent**
   ```bash
   sudo yum install -y amazon-cloudwatch-agent
   ```

2. **Configure Agent**
   ```json
   {
     "agent": {
       "metrics_collection_interval": 60,
       "run_as_user": "cwagent"
     },
     "metrics": {
       "namespace": "MyCustomMetrics",
       "metrics_collected": {
         "cpu": {
           "measurement": ["usage_idle", "usage_system"]
         },
         "mem": {
           "measurement": ["used_percent"]
         }
       }
     }
   }
   ```

3. **Start Agent**
   ```bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:config.json
   ```

## 9. Serverless Application

### Simple AWS Lambda with API Gateway

1. **Create Lambda Function**
   - Go to AWS Lambda
   - Create function
   - Author from scratch
   - Name: my-function
   - Runtime: Node.js 14.x
   - Create function

2. **Function Code**
   ```javascript
   exports.handler = async (event) => {
       const response = {
           statusCode: 200,
           body: JSON.stringify('Hello from Lambda!'),
       };
       return response;
   };
   ```

3. **Create API Gateway**
   - Go to API Gateway
   - Create API > REST API
   - Create Resource and Method (GET)
   - Deploy API

## 10. Multi-Tier Architecture

### Architecture Components:
- VPC with public and private subnets
- Application Load Balancer
- Auto Scaling Group
- RDS Database
- Security Groups

### Implementation Steps:
1. Create VPC with public and private subnets
2. Set up NAT Gateway for private subnets
3. Create Application Load Balancer
4. Configure Auto Scaling Group
5. Set up RDS in private subnets
6. Configure security groups

## 11. Kubernetes Cluster on AWS

### EKS Cluster Setup

1. **Install eksctl**
   ```bash
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   ```

2. **Create Cluster**
   ```bash
   eksctl create cluster \
   --name my-cluster \
   --version 1.21 \
   --region us-west-2 \
   --nodegroup-name standard-workers \
   --node-type t3.medium \
   --nodes 2 \
   --nodes-min 1 \
   --nodes-max 4
   ```

## 12. Security Hardening and Compliance

### Security Best Practices:

1. **IAM**
   - Use IAM roles instead of access keys
   - Implement least privilege
   - Enable MFA for all users

2. **Network Security**
   - Use security groups and NACLs
   - Implement VPC Flow Logs
   - Use AWS WAF and Shield

3. **Encryption**
   - Enable EBS encryption
   - Use KMS for managing keys
   - Enable SSL/TLS for all communications

## 13. Bash Scripting Challenges

### Log Analyzer Script
```bash
#!/bin/bash

# Log Analyzer
LOG_FILE="/var/log/syslog"
OUTPUT_FILE="log_analysis_$(date +%Y%m%d).txt"

# Count total lines
echo "Total log entries: $(wc -l < "$LOG_FILE")" > "$OUTPUT_FILE"

# Count errors
echo -e "\nError count: $(grep -i error "$LOG_FILE" | wc -l)" >> "$OUTPUT_FILE"

# Top 5 error messages
echo -e "\nTop 5 error messages:" >> "$OUTPUT_FILE"
grep -i error "$LOG_FILE" | sort | uniq -c | sort -nr | head -5 >> "$OUTPUT_FILE"

echo "Analysis complete. Results saved to $OUTPUT_FILE"
```

## 14. Linux Server Hardening

### Hardening Steps:

1. **Update System**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **SSH Hardening**
   ```bash
   # Edit SSH config
   sudo nano /etc/ssh/sshd_config
   
   # Recommended settings:
   # Port 2222  # Change from default 22
   # PermitRootLogin no
   # PasswordAuthentication no
   # MaxAuthTries 3
   # LoginGraceTime 60
   
   # Restart SSH
   sudo systemctl restart sshd
   ```

3. **Firewall Setup**
   ```bash
   sudo ufw allow 2222/tcp  # Your SSH port
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw enable
   ```

## 15. Build a Custom Linux Distribution

### Using Linux From Scratch (LFS)

1. **Prerequisites**
   ```bash
   sudo apt-get install -y build-essential bison gawk texinfo
   ```

2. **Download LFS Book**
   ```bash
   wget http://www.linuxfromscratch.org/lfs/downloads/stable/LFS-BOOK-11.1-NOCHUNKS.html
   ```

3. **Follow LFS Book**
   - The process is extensive and requires following the book step by step
   - It covers building a complete Linux system from source
   - Includes compiling the kernel, setting up the bootloader, and configuring the system

## Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [Linux From Scratch](https://www.linuxfromscratch.org/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Terraform Documentation](https://www.terraform.io/docs/index.html)

## Notes
- Always replace placeholders (like instance IDs, bucket names) with your actual values
- Ensure you understand the security implications of each configuration
- Regularly back up your work
- Follow the principle of least privilege when setting up IAM roles and permissions

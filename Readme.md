# EC2 Web Deployment with Ansible Automation

A complete guide to deploying a scalable web application on AWS EC2 instances with Ansible automation, Application Load Balancer, and comparison with S3 static hosting.

## 📋 Project Overview

This project demonstrates:
- Infrastructure as Code using Ansible
- High availability with Application Load Balancer
- Automated deployment across multiple EC2 instances
- Comparison between compute-based (EC2) and storage-based (S3) hosting
- AWS networking best practices

## 🏗️ Architecture
    Internet
        ↓
    Application Load Balancer (ALB)
        ↓
    [EC2 Instance 1]  [EC2 Instance 2]
    (AZ-1a)           (AZ-1b)
        ↓                  ↓
    NGINX              NGINX
        ↓                  ↓
    HTML Page        HTML Page

## 🛠️ Technologies Used

- **Cloud Platform:** AWS (VPC, EC2, ALB, S3)
- **Web Server:** NGINX
- **Automation:** Ansible
- **Version Control:** Git/GitHub
- **Languages:** HTML, CSS, JavaScript, Bash

## 📁 Repository Structure

    ec2-static-website/
    ├── README.md
    ├── index.html
    ├── instance-info.json (template)
    └── ansible/
        ├── inventory.ini
        ├── deploy-website.yml
        └── ansible.cfg


## 🚀 Prerequisites

## Local Machine Requirements
- Python 3.8+
- Ansible 2.9+
- AWS CLI
- Git
- SSH client

## AWS Requirements
- AWS Free Tier account
- IAM user with EC2, VPC, S3, and ALB permissions
- EC2 key pair created

## 📝 Setup Instructions

### 1. Clone the Repository


```bash
git clone https://github.com/YOUR_USERNAME/ec2-static-website.git
cd ec2-static-website
```

### 2. AWS Infrastructure Setup

#### Create VPC

*   **CIDR Block:** 10.0.0.0/16
    
*   **Name:** ec2-project-vpc
    

#### Create Subnets (in 2 different AZs)

*   **Subnet 1:** 10.0.1.0/24 (us-east-1a)
    
*   **Subnet 2:** 10.0.2.0/24 (us-east-1b)
    

#### Create Internet Gateway

*   Attach to VPC
    

#### Configure Route Tables

*   Route: 0.0.0.0/0 → Internet Gateway
    
*   Associate with both subnets
    

#### Security Groups

**EC2 Security Group (ec2-web-sg):**

```bash
Inbound Rules:
- SSH (22) from Your IP
- HTTP (80) from ALB Security Group
```

**ALB Security Group (alb-sg):**

```bash
Inbound Rules:
- HTTP (80) from 0.0.0.0/0
- HTTPS (443) from 0.0.0.0/0 `
```

### 3\. Launch EC2 Instances

Launch 2 t2.micro instances:

*   **AMI:** Amazon Linux 2023
    
*   **Instance Type:** t2.micro (Free Tier)
    
*   **Key Pair:** Your EC2 key pair
    
*   **Network:** Place in different subnets (different AZs)
    
*   **Auto-assign Public IP:** Enabled
    
*   **Security Group:** ec2-web-sg
    

### 4\. Configure Ansible

#### Update Inventory File

Edit ansible/inventory.ini:

ini
```bash
[webservers]
web-server-1 ansible_host=<EC2_PUBLIC_IP_1> ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/ec2-project-key.pem
web-server-2 ansible_host=<EC2_PUBLIC_IP_2> ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/ec2-project-key.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_common_args='-o StrictHostKeyChecking=no'`
```

#### Update Playbook Variables

Edit ansible/deploy-website.yml and update:

yaml

vars:
 ```bash  github_repo: "https://github.com/YOUR_USERNAME/ec2-static-website.git"  ````

#### Test Connection

bash
```bash
cd ansible
ansible -i inventory.ini webservers -m ping
```
Expected output:
```bash
web-server-1 | SUCCESS => { ... }
web-server-2 | SUCCESS => { ... }`
```

### 5\. Deploy with Ansible

bash

ansible-playbook -i inventory.ini deploy-website.yml

This playbook will:

*   ✅ Update system packages
    
*   ✅ Install NGINX
    
*   ✅ Configure NGINX to start on boot
    
*   ✅ Clone website from GitHub
    
*   ✅ Generate instance-specific info (IP and Instance ID)
    
*   ✅ Configure NGINX virtual host
    

### 6\. Setup Application Load Balancer

#### Create Target Group

*   **Name:** ec2-web-tg
    
*   **Protocol:** HTTP:80
    
*   **Health Check Path:** /
    
*   **Targets:** Both EC2 instances
    

#### Create ALB

*   **Name:** ec2-web-alb
    
*   **Scheme:** Internet-facing
    
*   **Subnets:** Both subnets (multi-AZ)
    
*   **Security Group:** alb-sg
    
*   **Listener:** HTTP:80 → ec2-web-tg
    

#### Update Security Group

Edit ec2-web-sg to only allow HTTP from ALB:

Remove: HTTP (80) from 0.0.0.0/0
Add: HTTP (80) from alb-sg`

### 7\. Test Deployment

Access your website via ALB DNS:

http://your-alb-dns-name-xxxx.region.elb.amazonaws.com

Refresh the page multiple times - you should see different instance IPs and IDs, demonstrating load balancing.

🪣 S3 Static Website Hosting
----------------------------

### Setup S3 Bucket

1.  Create S3 bucket with public access
    
2.  Enable static website hosting
    
3.  Upload index.html and instance-info.json
    
4.  Add bucket policy for public read access
    

### Bucket Policy
```bash
json

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
    ]
}  `
```

### Access S3 Website
```bash
http://your-bucket-name.s3-website-region.amazonaws.com
```

🔄 EC2 vs S3 Comparison
-----------------------

# EC2 vs S3 Comparison

  -------------------------------------------------------------------------
  Feature           EC2 + ALB                    S3 Static Hosting
  ----------------- ---------------------------- --------------------------
  **Cost**          Higher (compute + storage)   Lower (storage only)

  **Scalability**   Manual/Auto-scaling          Automatic unlimited

  **Maintenance**   Requires patching, updates   Zero maintenance

  **Dynamic         ✅ Yes                       ❌ Static only
  Content**                                      

  **Compute Power** ✅ Full control              ❌ No compute

  **SSL/CDN**       Manual setup                 Easy with CloudFront

  **Best For**      Web apps, APIs, databases    Static sites, SPAs
  -------------------------------------------------------------------------

    

🧹 Cleanup (IMPORTANT!)
-----------------------

**Delete resources in this order to avoid charges:**

bash

``` bash
# 1. Delete ALB
aws elbv2 delete-load-balancer --load-balancer-arn <ALB_ARN>
aws elbv2 delete-target-group --target-group-arn <TG_ARN>

# 2. Terminate EC2 Instances
aws ec2 terminate-instances --instance-ids <INSTANCE_ID_1> <INSTANCE_ID_2>

# 3. Empty and delete S3 bucket
aws s3 rm s3://your-bucket-name --recursive
aws s3 rb s3://your-bucket-name

# 4. Delete network resources (via Console or CLI)
```

Or use AWS Console:

1.  Delete Load Balancer and Target Group
    
2.  Terminate EC2 instances
    
3.  Empty and delete S3 bucket
    
4.  Delete NAT Gateway, Elastic IPs
    
5.  Detach and delete Internet Gateway
    
6.  Delete subnets
    
7.  Delete VPC (this deletes route tables and security groups)
    


🐛 Troubleshooting
------------------

### Ansible Connection Failed

bash
```bash
# Check SSH access
ssh -i ~/.ssh/ec2-project-key.pem ec2-user@<PUBLIC_IP>

# Verify security group allows SSH from your IP```
```
### Website Not Loading

bash

```bash
# Check NGINX status on EC2
sudo systemctl status nginx

# Check NGINX logs
sudo tail -f /var/log/nginx/error.log
```

### ALB Health Checks Failing

*   Verify security group allows HTTP from ALB to EC2
    
*   Check NGINX is running: sudo systemctl status nginx
    
*   Verify / path returns 200 OK
    

### Instance Metadata Not Showing

bash

```bash
# SSH into instance and test
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/local-ipv4
```

📚 Additional Resources
-----------------------

*   [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
    
*   [Ansible Documentation](https://docs.ansible.com/)
    
*   [AWS Free Tier Details](https://aws.amazon.com/free/)
    
*   [NGINX Documentation](https://nginx.org/en/docs/)
    


👤 Author
---------

**Your Name**

*   GitHub: [@Amarachi32](https://github.com/Amarachi32)
    
*   Medium: [@prptamarachi](https://medium.com/@prptamarachi/deploying-a-scalable-web-application-on-aws-a-hands-on-journey-from-ec2-to-load-balancers-679e52ca6f83)
*   Google Docs: for Image reference:  (https://docs.google.com/document/d/1hNGVsFw9butB6EbjF_4Ne_KruIdosZ70p74xt5P0WOs/edit?usp=sharing)
    

🤝 Contributing
---------------

This is a learning project. Feel free to fork and experiment!

⚠️ Important Notes
------------------

1.  **No Public IP Screenshots:** Always access via ALB DNS, never direct EC2 IP
    
2.  **Free Tier Limits:** 750 hours/month for t2.micro (2 instances = 375 hours each)
    
3.  **Delete Resources:** Avoid unexpected charges by cleaning up after completion
    
4.  **Security:** Never commit AWS credentials or private keys to GitHub
    
5.  **Billing Alerts:** Set up AWS billing alerts for $5-10
    

**Happy Learning! 🚀**

_Remember: The goal is not just to complete the project, but to understand WHY each component exists and HOW they work together._

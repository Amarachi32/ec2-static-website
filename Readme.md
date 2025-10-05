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

### Local Machine Requirements
- Python 3.8+
- Ansible 2.9+
- AWS CLI
- Git
- SSH client

### AWS Requirements
- AWS Free Tier account
- IAM user with EC2, VPC, S3, and ALB permissions
- EC2 key pair created

## 📝 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/ec2-static-website.git
cd ec2-static-website
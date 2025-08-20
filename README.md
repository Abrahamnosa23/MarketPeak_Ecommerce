# MarketPeak E-Commerce Platform Deployment

## Project Overview
This project outlines the end-to-end deployment of "MarketPeak," a functional e-commerce platform, onto AWS infrastructure. The implementation emphasizes core DevOps principles: Infrastructure as Code (IaC) concepts, version control with Git, automated deployment workflows, and continuous integration/delivery practices.

## Architecture & Workflow

-![Architecture & Workflow.png](https://github.com/Abrahamnosa23/MarketPeak_Ecommerce/blob/main/Architecture%20%26%20Workflow/Architecture%20%26%20Workflow.png)

## Prerequisites
- AWS Account with permissions to launch EC2 instances.
- Git installed locally.
- SSH Client (OpenSSH, PuTTY).
- GitHub Account.
- Basic familiarity with the Linux command line.

## Phase 1: Version Control Implementation

### 1.1 Local Repository Setup
```bash
# Initialize the project directory and Git repository
mkdir MarketPeak_Ecommerce
cd MarketPeak_Ecommerce
git init
```
-![1.png](https://github.com/Abrahamnosa23/MarketPeak_Ecommerce/blob/main/Screenshot/Phase1_Screenshots/1.png)

### 1.2 Template Acquisition and Preparation
- Sourced a production-ready e-commerce HTML template from Topplate.
- Extracted the template contents into the MarketPeak_Ecommerce directory.
- Performed basic branding customizations (updated site title, logo, color scheme in index.html and css/style.css).

-![02.png](https://github.com/Abrahamnosa23/MarketPeak_Ecommerce/blob/main/Screenshot/Phase1_Screenshots/02.png)

### 1.3 Initial Commit
```bash
# Configure Git user (first time only)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Stage and commit the base template
git add .
git commit -m "feat: initial commit with base e-commerce template structure"
```

### 1.4 GitHub Repository Synchronization
- Created a new public repository on GitHub named MarketPeak_Ecommerce (no README, .gitignore, or license).
- Added the remote origin and pushed the code.
```bash
git remote add origin https://github.com/YourUsername/MarketPeak_Ecommerce.git
git branch -M main
git push -u origin main
```

## Phase 2: AWS Infrastructure and Deployment

### 2.1 EC2 Instance Provisioning
- Service: AWS EC2
- AMI: Amazon Linux 2023 AMI
- Instance Type: t2.micro (Free Tier eligible)
- Security Group Rules:
    - Inbound: SSH (22), HTTP (80)
    - Outbound: All traffic (0.0.0.0/0)
- Key Pair: Created and downloaded a new .pem key pair for SSH access.

### 2.2 Server Access and Repository Cloning
SSH into the instance using the private key.
```bash
ssh -i "~/path/to/your-key.pem" ec2-user@<YOUR-EC2-PUBLIC-IP>
```
On the EC2 instance, clone the project repository using HTTPS:
```bash
sudo yum update -y
git clone https://github.com/YourUsername/MarketPeak_Ecommerce.git
```

### 2.3 Web Server Installation and Configuration
Installed the Apache HTTP Server (httpd).
```bash
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

### 2.4 Application Deployment
Copied the application files to Apache's document root.
```bash
# Remove default test page
sudo rm -rf /var/www/html/*

# Deploy MarketPeak application files
sudo cp -r MarketPeak_Ecommerce/* /var/www/html/

# Apply correct read/execute permissions
sudo chmod -R 755 /var/www/html
```

### 2.5 Validation
Accessed the EC2 instance's public IP address in a web browser to confirm the MarketPeak website was live and functional.

## Phase 3: CI/CD Workflow Implementation
Development Branch Strategy
```bash
# Create and switch to a feature branch
git checkout -b feat/new-product-carousel

# ... make changes ...
git add .
git commit -m "feat: add responsive product carousel to homepage"

# Push branch to remote
git push origin feat/new-product-carousel
```

## Pull Request & Merge
- Pushed feature branch to GitHub.
- Created a Pull Request (PR) from feat/new-product-carousel into main.
- Performed a code review (simulated or actual).
- Merged the PR upon approval.

## Production Deployment
SSH into the production server and pull the latest merged code.
```bash
cd /var/www/html
sudo git pull origin main
# Restart httpd if necessary (e.g., for config changes)
sudo systemctl restart httpd
```

## Troubleshooting & Issue Resolution
Issue 1: Apache Default "It Works!" Page is Displayed
  - Problem: The browser showed the default Apache test page instead of the MarketPeak website.
  - Root Cause: Application files were not correctly placed in, or permissions were insufficient for, the /var/www/html/ directory.
  - Solution:
      - Verified the files were copied correctly: ls -la /var/www/html/
      - Recopied the files with the correct command: sudo cp -r ~/MarketPeak_Ecommerce/* /var/www/html/
      - Ensured correct permissions: sudo chmod -R 755 /var/www/html
      - Restarted the web server: sudo systemctl restart httpd

Issue 2: Permission Denied (Publickey) on SSH Login
    - Problem: Could not SSH into the EC2 instance.
    - Root Cause: Incorrect permissions on the private key file or wrong username.
    - Solution:
        - Set strict permissions on the key: chmod 400 your-key.pem
        - Confirmed the correct SSH user (ec2-user for Amazon Linux, ubuntu for Ubuntu AMIs).

Issue 3: HTTP Connection Timed Out (Port 80)
  - Problem: Could not access the website via HTTP.
  - Root Cause: EC2 Security Group was not configured to allow inbound traffic on port 80.
  - Solution: Edited the Security Group inbound rules to allow HTTP (80) from source 0.0.0.0/0.

Issue 4: Git Pull Error on Production Server (Permission Denied)
  - Problem: Could not run git pull on the server due to permission errors in the /var/www/html directory.
  - Root Cause: The files were owned by the root user due to use of sudo cp.
  - Solution: For a more robust CI/CD pipeline, the recommended practice is to:
      - Clone the repository directly into /var/www/html (owned by ec2-user).
      - Use a post-merge hook or a dedicated deployment script to adjust file ownership for Apache: sudo chown -R apache:apache /var/www/html after pulling.

## Best Practices Implemented
  - Version Control: All infrastructure code and application assets are versioned in Git.
  - Branching Strategy: Used feature branches and pull requests to isolate changes and facilitate code review.
  - Infrastructure as Code (IaC): Manual server setup was documented to be easily reproducible. The next evolution would be to use Terraform or AWS CloudFormation to script the EC2 provisioning.
  - Documentation: This README serves as a comprehensive runbook for the entire deployment process.
  - Security:
      - Used SSH key pairs for secure access.
      - Minimal security group rules following the principle of least privilege.
  - Automation: Established a manual CI/CD pipeline. The process can be automated using GitHub Actions or Jenkins to trigger deployments on a push to the main branch.

## Future Enhancements
- Automated Provisioning: Implement infrastructure provisioning using Terraform.
- CI/CD Automation: Integrate GitHub Actions to automatically test, build, and deploy upon a merge to main.
- Configuration Management: Use Ansible or Puppet to manage server configuration and application deployment.
- Containerization: Dockerize the application for consistent environments across development, testing, and production.
- Reverse Proxy & SSL: Place Apache behind an Nginx reverse proxy and install a TLS/SSL certificate using Let's Encrypt for HTTPS.

## License
This project is for educational purposes as part of a DevOps capstone project.

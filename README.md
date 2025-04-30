# 🚀 Deploy a Static Website on EC2 Using AWS CI/CD Services (CodeBuild, CodeDeploy, GitHub, and Nginx)

This project demonstrates how to set up a **CI/CD pipeline on AWS** to deploy a static website to an **EC2 Ubuntu instance** using:

- **GitHub** – Source Code Management
- **AWS CodeBuild** – Build and package the project
- **Amazon S3** – Store the build artifacts
- **AWS CodeDeploy** – Deploy the application to EC2
- **Nginx** – Web server to serve static content

---

## 📁 Repository Structure

```
AWS-Code-Service-Blog/
├── appspec.yml         # Deployment instructions for CodeDeploy
├── buildspec.yml       # Build instructions for CodeBuild
├── install_nginx.sh    # Hook script to install Nginx on EC2
├── start_nginx.sh      # Hook script to start Nginx on EC2
└── index.html          # Static HTML file to be deployed
```

---

## ⚙️ CI/CD Workflow Overview

1. Source code is hosted on **GitHub**.
2. **CodeBuild** fetches the code, installs Nginx, moves `index.html` to `/var/www/html`, and uploads artifacts to **S3**.
3. **CodeDeploy** pulls the artifacts and deploys them on an **EC2 Ubuntu instance** running **Nginx**.
4. You access the website using the EC2 public IP.

---

## 📝 Prerequisites

- AWS Account
- IAM User with full access to:
  - EC2
  - CodeBuild
  - CodeDeploy
  - S3
- Basic knowledge of AWS services
- GitHub account

---

## 🔧 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/Pravesh-Sudha/AWS-Code-Service-Blog.git
cd AWS-Code-Service-Blog
```

### 2. Create an S3 Bucket

- Go to S3 > Create Bucket
- Name it something unique (e.g., `demo-1231234`)
- Leave defaults and create the bucket

### 3. Configure CodeBuild

- Go to CodeBuild > Create Build Project
- Project Name: `Nginx-Build`
- Source Provider: GitHub
- Repo URL: `https://github.com/Pravesh-Sudha/AWS-Code-Service-Blog`
- OS: Ubuntu
- Buildspec: Use buildspec.yml
- Artifacts:
  - Type: Amazon S3
  - Bucket: `demo-1231234`

Start the build and wait for it to complete (~4-5 minutes).

### 4. IAM Roles

#### EC2 Role: `ec2-code-deploy`

- IAM > Roles > Create Role
- Use Case: EC2
- Permissions:
  - `AmazonEC2FullAccess`
  - `AmazonS3FullAccess`
  - `AWSCodeDeployFullAccess`
  - `AmazonEC2RoleforAWSCodeDeploy`
  - `AmazonEC2RoleforAWSCodeDeployLimited`
  - `AWSCodeDeployRole`

#### CodeDeploy Role: `ec2-code-deploy-role`

- Use Case: CodeDeploy
- Permissions:
  - `AWSCodeDeployRole`
  - `AmazonS3FullAccess`
  - `AmazonEC2FullAccess`

---

### 5. Launch EC2 Instance

- Launch a `t2.micro` Ubuntu instance
- Allow HTTP and HTTPS in security group
- Once launched, connect via EC2 Instance Connect

#### Install CodeDeploy Agent

```bash
sudo apt update
sudo apt install ruby-full -y
sudo apt install wget -y
cd /home/ubuntu
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo systemctl status codedeploy-agent
```

> ⚠️ Make sure to replace the region in the `wget` URL if you're using a region other than `us-east-1`.

#### Attach IAM Role

- Go to EC2 > Select Instance > Actions > Security > Modify IAM Role
- Attach the `ec2-code-deploy` role

---

### 6. Configure CodeDeploy

- CodeDeploy > Create Application
  - Name: `nginx-app`
  - Platform: EC2/on-premises

- Create Deployment Group
  - Name: `nginx-deploy-grp`
  - Service Role: `ec2-code-deploy-role`
  - EC2 Tag: `Name=nginx-demo`
  - Install Agent: Never (already done manually)
  - Load Balancing: Disabled

---

### 7. Create Deployment

- CodeDeploy > Select `nginx-app` > Select `nginx-deploy-grp` > Create Deployment
- Revision Type: My application is stored in Amazon S3
- Revision Location: `s3://demo-1231234/Nginx-build`
- File type: `.zip`
- Click **Create Deployment**

---

## 🌐 Access Your Website

Once deployment is successful, visit the **public IP** of your EC2 instance in a browser.  
You should see the `index.html` page served by **Nginx**!

---

## 📚 Related Blog

🔗 [Deploy a Static Website on EC2 using AWS CI/CD Services – Full Guide on Dev.to](https://dev.to/aws-builders/deploy-a-static-website-on-ec2-using-aws-cicd-services-codebuild-codedeploy-github-and-nginx-5bkl)

---

## 🙌 Connect With Me

If you found this helpful, feel free to follow me on:

- 💼 [LinkedIn](https://www.linkedin.com/in/pravesh-sudha)
- 🐦 [Twitter (X)](https://twitter.com/PraveshSudha)
- 📝 [Dev.to](https://dev.to/praveshsudha)

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

Happy Deploying! 🚀
```

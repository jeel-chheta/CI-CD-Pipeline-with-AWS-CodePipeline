# CI/CD Pipeline with AWS CodePipeline

## 📌 Project Overview
This project sets up a **CI/CD pipeline** using AWS native services to automate the deployment of a web application. The pipeline integrates **GitHub, AWS CodePipeline, AWS CodeBuild, AWS CodeDeploy, S3, and EC2** to ensure a smooth and automated deployment process.

## 🚀 Tech Stack
- **AWS CodePipeline** - CI/CD pipeline service
- **AWS CodeBuild** - Compiles and tests the application
- **AWS CodeDeploy** - Deploys the application to EC2
- **AWS S3** - Stores build artifacts
- **AWS EC2** - Hosts the application
- **GitHub** - Version control system

---

## 📂 Project Structure
```
├── app/
│   ├── index.html
│
├── scripts/
│   ├── restart_server.sh
│
├── buildspec.yml
├── appspec.yml
├── README.md
```

---

## 🛠️ Setup Instructions

### Step 1: Set Up IAM Role for CodePipeline
1. Go to **AWS IAM** → Create a new role.
2. Choose **AWS Service** → Select **CodePipeline**.
3. Attach the following policies:
   - `AmazonS3FullAccess`
   - `AWSCodeBuildAdminAccess`
   - `AWSCodeDeployRole`
4. Name the role **CodePipelineRole** and save it.

### Step 2: Create a GitHub Repository
1. Create a GitHub repository (e.g., `my-ci-cd-project`).
2. Clone the repository:
   ```sh
   git clone https://github.com/jeel-chheta/CI-CD-Pipeline-with-AWS-CodePipeline.git
   cd CI-CD-Pipeline-with-AWS-CodePipeline
   ```
3. Add a sample application:
   ```sh
   mkdir app && cd app
   echo "Hello, AWS CI/CD!" > index.html
   ```
4. Commit and push the code:
   ```sh
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

### Step 3: Create an S3 Bucket for Artifacts
1. Go to **AWS S3** → Create a new bucket (`my-cicd-artifacts`).
2. Enable versioning for tracking changes.

### Step 4: Set Up AWS CodeBuild
1. Go to **AWS CodeBuild** → Create a new project.
2. Select **GitHub** as the source and connect your repository.
3. Use **Amazon Linux** as the environment.
4. Add the `buildspec.yml` file:
   ```yaml
   version: 0.2
   phases:
     install:
       runtime-versions:
         nodejs: latest
     build:
       commands:
         - echo "Building application..."
         - mkdir output
         - cp index.html output/
   artifacts:
     files:
       - output/*
   ```

### Step 5: Set Up AWS CodeDeploy
1. **Create IAM Role for EC2:**
   - Create a new IAM role for EC2.
   - Attach **AWSCodeDeployRole** policy.
   - Assign it to your EC2 instance.

2. **Install CodeDeploy Agent on EC2:**
   ```sh
   sudo yum update -y
   sudo yum install ruby -y
   sudo yum install wget -y
   cd /home/ec2-user
   wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
   chmod +x ./install
   sudo ./install auto
   sudo service codedeploy-agent start
   sudo systemctl enable codedeploy-agent
   ```

3. **Create a CodeDeploy Application:**
   - Go to **AWS CodeDeploy** → Create an application.
   - Choose **EC2/On-Premises** as the compute platform.

4. **Add Deployment Files:**
   - `appspec.yml`:
     ```yaml
     version: 0.0
     os: linux
     files:
       - source: /
         destination: /var/www/html/
     hooks:
       AfterInstall:
         - location: scripts/restart_server.sh
           timeout: 300
           runas: root
     ```
   - `scripts/restart_server.sh`:
     ```sh
     #!/bin/bash
     systemctl restart httpd
     ```

5. Push changes to GitHub:
   ```sh
   git add .
   git commit -m "Added deployment files"
   git push origin main
   ```

### Step 6: Set Up AWS CodePipeline
1. Go to **AWS CodePipeline** → Create a new pipeline.
2. Select **GitHub** as the source.
3. Choose **AWS CodeBuild** as the build stage.
4. Select **AWS CodeDeploy** as the deployment stage.
5. Click **Create Pipeline**.

### Step 7: Deploy and Verify
1. Make a change to `index.html`:
   ```sh
   echo "Updated version!" > index.html
   git add .
   git commit -m "Updated website content"
   git push origin main
   ```
2. **AWS CodePipeline** will automatically:
   - Pull the updated code from GitHub.
   - Build the artifacts.
   - Deploy them to EC2.
3. Open **http://your-ec2-public-ip** in a browser.
   - You should see **"Updated version!"**.

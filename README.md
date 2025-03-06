# Codepipeline-using-Jenkins-and-SonarQube
# Jenkins and SonarQube Integration Guide

This guide provides instructions for setting up Jenkins and SonarQube on Ubuntu EC2 instances, integrating them, and configuring a pipeline for automated code analysis.

---

## Prerequisites
- Two EC2 instances (Ubuntu, `t2.medium`, key pair, and security group with necessary ports open).
- Access to the AWS Management Console.
- Basic knowledge of Jenkins, SonarQube, and GitHub.

---

## Steps to Set Up Jenkins and SonarQube

### 1. Launch EC2 Instances
1. Launch two EC2 instances with the following details:
   - **Instance Type**: `t2.medium`
   - **OS**: Ubuntu
   - **Key Pair**: Use your existing key pair.
   - **Security Group**: Ensure ports `8080` (Jenkins) and `9000` (SonarQube) are open.
   - **Instance Names**:
     - `ID_practicejenkins` (for Jenkins)
     - `ID_practicesonarqube` (for SonarQube)
2. Note the **Public IPs** of both instances.

---

### 2. Install and Configure Jenkins
1. Connect to the Jenkins server using **EC2 Instance Connect**.
2. Run the following commands to install Jenkins:

   ```bash
   sudo apt update
   sudo apt install openjdk-17-jre
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
     https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   systemctl status jenkins
   ```

3. Copy the **initial admin password** from the logs:
   ```
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
4. Open a browser and navigate to:
   ```
   http://<Jenkins_Public_IP>:8080
   ```
5. Paste the initial admin password and complete the setup:
   - Create an admin user (note the username and password).
   - Install suggested plugins.

---

### 3. Create a Jenkins Pipeline
1. On the Jenkins homepage, click **New Item**.
2. Select **Pipeline** and provide a name.
3. Under **Source Code Management**, select **Git** and provide your GitHub repository URL.
4. Enable **GitHub hook trigger for GITScm polling**.
5. Save the pipeline.

---

### 4. Configure GitHub Webhook
1. Go to your GitHub repository.
2. Navigate to **Settings > Webhooks**.
3. Add a new webhook:
   - **Payload URL**: `http://<Jenkins_Public_IP>:8080/github-webhook/`
   - **Content Type**: `application/json`
   - **Events**: Select **Pull requests** and **Pushes**.
4. Save the webhook.

---

### 5. Install and Configure SonarQube
1. Connect to the SonarQube server using **EC2 Instance Connect**.
2. Run the following commands to install SonarQube:

   ```bash
   sudo apt update
   sudo apt install openjdk-17-jre
   wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.1.0.102122.zip
   sudo apt install unzip
   unzip sonarqube-25.1.0.102122.zip
   cd sonarqube-25.1.0.102122/bin/linux-x86-64
   ./sonar.sh console
   ```

3. Open a browser and navigate to:
   ```
   http://<SonarQube_Public_IP>:9000
   ```
4. Log in with the default credentials:
   - **Username**: `admin`
   - **Password**: `admin`
5. Change the password to `Welcome@1234`.
6. Create a new project manually:
   - Provide a project name (e.g., `YourID`).
   - Select **Analysis Method with Jenkins**.
   - Copy the **Project Key** (e.g., `sonar.projectKey=A10001`).

---

### 6. Generate SonarQube Token
1. Go to your user profile in SonarQube.
2. Navigate to **Security > Tokens**.
3. Generate a new token:
   - **Name**: `Global`
   - **Expiration**: 30 days
4. Copy the token (e.g., `sqa_d8da872ebf47966d06a29d704388f60d9bc9ff8b`).

---

### 7. Integrate SonarQube with Jenkins
1. In Jenkins, go to **Manage Jenkins > Manage Plugins**.
2. Install the following plugins:
   - **SonarQube Scanner**
   - **SSH2 Easy**
3. Configure SonarQube in Jenkins:
   - Go to **Manage Jenkins > Configure System**.
   - Under **SonarQube Servers**, add a new server:
     - **Name**: `SonarQube`
     - **URL**: `http://<SonarQube_Public_IP>:9000`
   - Under **Server Authentication**, add a new secret text:
     - **Kind**: `Secret text`
     - **Secret**: Paste the SonarQube token.
   - Save the configuration.

---

### 8. Configure Jenkins Pipeline for SonarQube
1. Go to your Jenkins pipeline and click **Configure**.
2. Under **Build Steps**, add a new step:
   - **Execute SonarQube Scanner**.
3. Paste the **Analysis Properties** (e.g., `sonar.projectKey=A10001`).
4. Save the pipeline.

---

### 9. Run the Pipeline
1. Trigger the Jenkins pipeline manually or via a GitHub push.
2. Verify the build status and check the SonarQube dashboard for analysis results.

---

## Notes
- Ensure the security group allows inbound traffic on ports `8080` (Jenkins) and `9000` (SonarQube).
- Use the correct project key and token for SonarQube integration.
- Monitor Jenkins and SonarQube logs for any errors.

---

## Troubleshooting
- If Jenkins or SonarQube fails to start:
  - Check the service status using `systemctl status jenkins` or `./sonar.sh status`.
  - Verify the security group rules.
- If the pipeline fails:
  - Check the Jenkins console output for errors.
  - Ensure the GitHub webhook is configured correctly.

---

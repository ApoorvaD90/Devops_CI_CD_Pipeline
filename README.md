# Flask CI/CD Pipeline

**Prepared by:** Apoorva Deshpande
**Email:** deshpande.apoorva123@gmail.com

---

## Project Overview

This project demonstrates two CI/CD pipeline implementations for a simple Python Flask web application:

1. **Part 1:** Jenkins CI/CD Pipeline on AWS EC2
2. **Part 2:** GitHub Actions CI/CD Pipeline

---

## Project Structure

```
Devops_CI_CD_Pipeline/
├── app.py                        # Main Flask application
├── test_app.py                   # Unit tests (pytest)
├── requirements.txt              # Python dependencies
├── Jenkinsfile                   # Jenkins pipeline definition
├── .github/
│   └── workflows/
│       └── ci-cd.yml             # GitHub Actions workflow
└── README.md                     # This file
```

## Flask Application

### API Endpoints

| Endpoint     | Method | Description                |
|--------------|--------|----------------------------|
| /            | GET    | Returns welcome message    |
| /health      | GET    | Health check endpoint      |
| /add/{a}/{b} | GET    | Returns sum of two numbers |

### Running Locally

```bash
pip install -r requirements.txt
python app.py
# Visit http://localhost:5000
```

### Running Tests Locally

```bash
pytest test_app.py -v
```

---

# Part 1: Jenkins CI/CD Pipeline

## Objective

Set up a Jenkins pipeline that automates the testing and deployment of a simple Python Flask web application.

## Prerequisites

- AWS Account with EC2 access
- GitHub account
- Gmail account with App Password for notifications
- Python 3.x
- Jenkins 2.x

## Setup Instructions

### 1. Launch AWS EC2 Instance

1. Launch an EC2 instance with the following settings:
   - **AMI:** Ubuntu Server 22.04 LTS
   - **Instance type:** t3.micro
   - **Storage:** 20 GB
2. Configure Security Group inbound rules:

| Port | Protocol | Source    | Purpose    |
|------|----------|-----------|------------|
| 22   | TCP      | My IP     | SSH access |
| 8080 | TCP      | 0.0.0.0/0 | Jenkins UI |
| 5000 | TCP      | 0.0.0.0/0 | Flask app  |

### 2. Install Jenkins on EC2

SSH into the EC2 instance and run:

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y openjdk-21-jdk

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins

sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### 3. Install Python on Jenkins Server

```bash
sudo apt install -y python3 python3-pip python3.14-venv
```

### 4. Initial Jenkins Setup

1. Open browser and navigate to `http://<EC2-IP>:8080`
2. Get the initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
3. Select **Install suggested plugins**
4. Create admin user
5. Save and start using Jenkins

### 5. Install Required Jenkins Plugins

Go to **Manage Jenkins → Plugins → Available Plugins** and install:

| Plugin                    | Purpose                             |
|---------------------------|-------------------------------------|
| GitHub Integration Plugin | Connects Jenkins to GitHub webhooks |
| Pipeline                  | Enables Jenkinsfile-based pipelines |
| Email Extension Plugin    | Sends build notifications via email |
| Git Plugin                | Git source code management          |

### 6. Configure Email Notifications

Go to **Manage Jenkins → System → E-mail Notification**:

| Field                   | Value               |
|-------------------------|---------------------|
| SMTP Server             | smtp.gmail.com      |
| Use SMTP Authentication | Checked             |
| Username                | your Gmail address  |
| Password                | Gmail App Password  |
| Use SSL                 | Checked             |
| SMTP Port               | 465                 |

> **Note:** Use a Gmail App Password, not your regular Gmail password.
> Generate one at: Google Account → Security → 2-Step Verification → App Passwords.

### 7. Connect Jenkins to GitHub

#### Generate GitHub Personal Access Token

1. Go to **GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)**
2. Click **Generate new token (classic)**
3. Select scopes: `repo` and `admin:repo_hook`
4. Copy the token immediately

#### Add Credentials to Jenkins

1. Go to **Manage Jenkins → Credentials → System → Global Credentials → Add Credentials**
2. Fill in:

| Field    | Value                  |
|----------|------------------------|
| Kind     | Username with password |
| Username | Your GitHub username   |
| Password | Your GitHub PAT token  |
| ID       | github-credentials     |

### 8. Create Jenkins Pipeline Job

1. Click **New Item** → enter name `flask-jenkins-pipeline` → select **Pipeline** → click **OK**
2. Under **General**: Check **GitHub project** → enter repository URL
3. Under **Build Triggers**: Check **GitHub hook trigger for GITScm polling**
4. Under **Pipeline**:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL: your GitHub repo URL
   - Credentials: **github-credentials**
   - Branch Specifier: `*/main`
   - Script Path: `Jenkinsfile`
5. Click **Save**

### 9. Set Up GitHub Webhook

1. Go to your GitHub repo → **Settings → Webhooks → Add webhook**
2. Fill in:

| Field         | Value                                  |
|---------------|----------------------------------------|
| Payload URL   | http://\<EC2-IP\>:8080/github-webhook/ |
| Content type  | application/json                       |
| Which events? | Just the push event                    |
| Active        | Checked                                |

## Jenkins Pipeline Stages

| Stage    | Description                                            |
|----------|--------------------------------------------------------|
| Checkout | Pulls latest source code from GitHub                   |
| Build    | Creates Python virtual environment, installs packages  |
| Test     | Runs pytest unit tests; pipeline fails if tests fail   |
| Deploy   | Deploys Flask app to staging server                    |

## Jenkins Notifications

The pipeline sends email notifications on:
- Build **SUCCESS**
- Build **FAILURE**

---

# Part 2: GitHub Actions CI/CD Pipeline

## Objective

Implement a CI/CD workflow using GitHub Actions for the Python Flask application.

## Prerequisites

- GitHub account
- AWS EC2 instance (Ubuntu 22.04, t3.micro)
- SSH key pair for EC2 access

## Setup Instructions

### 1. Create Staging Branch

```bash
git checkout -b staging
git push origin staging
```

### 2. Launch AWS EC2 Instance for Deployment

1. Launch an EC2 instance with the following settings:
   - **AMI:** Ubuntu Server 22.04 LTS
   - **Instance type:** t3.micro
   - **Storage:** 20 GB
2. Configure Security Group inbound rules:

| Port | Protocol | Source    | Purpose       |
|------|----------|-----------|---------------|
| 22   | TCP      | 0.0.0.0/0 | SSH (Actions) |
| 5000 | TCP      | 0.0.0.0/0 | Flask app     |

### 3. Set Up EC2 Server

SSH into the EC2 instance and run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3.14-venv git
git clone https://github.com/ApoorvaD90/Devops_CI_CD_Pipeline \
  /home/ubuntu/Devops_CI_CD_Pipeline
cd /home/ubuntu/Devops_CI_CD_Pipeline
pip install -r requirements.txt --break-system-packages
```

### 4. Configure GitHub Secrets

Go to **GitHub repo → Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret Name     | Value                                   |
|-----------------|-----------------------------------------|
| SSH_PRIVATE_KEY | Full contents of your EC2 .pem key file |
| STAGING_HOST    | EC2 public IP address                   |
| STAGING_USER    | ubuntu                                  |
| PROD_HOST       | EC2 public IP address                   |
| PROD_USER       | ubuntu                                  |

> **How to get SSH_PRIVATE_KEY:** Convert your .ppk file to .pem using PuTTYgen
> (Conversions → Export OpenSSH key). Open the .pem file in a text editor and
> copy the entire contents including the header and footer lines.

### 5. GitHub Actions Workflow

The workflow file is located at `.github/workflows/ci-cd.yml`

## Workflow Jobs

| Job                            | Trigger                  | Description                           |
|--------------------------------|--------------------------|---------------------------------------|
| Install Dependencies & Run Tests | All pushes             | Installs pip packages and runs pytest |
| Build                          | After tests pass         | Verifies app is ready for deployment  |
| Deploy to Staging              | Push to `staging` branch | SSHs into EC2 and deploys Flask app   |
| Deploy to Production           | Release tag (`v*`)       | SSHs into EC2 and deploys to production |

## Workflow Triggers

| Event               | Jobs Triggered                                                       |
|---------------------|----------------------------------------------------------------------|
| Push to `main`      | Install Dependencies → Run Tests → Build                             |
| Push to `staging`   | Install Dependencies → Run Tests → Build → Deploy to Staging         |
| Create tag `v1.0.0` | Install Dependencies → Run Tests → Build → Deploy to Production      |

## Testing the Workflow

### Test Staging Deployment

```bash
git checkout staging
git merge main
git push origin staging
```

Go to **GitHub → Actions** tab to watch the workflow run.
Verify the app is running at `http://<EC2-IP>:5000`

### Test Production Deployment

```bash
git checkout main
git tag v1.0.0
git push origin v1.0.0
```

Go to **GitHub → Actions** tab to watch the production deployment trigger.

## Environment Secrets

All sensitive information is stored as GitHub Secrets:

- **SSH_PRIVATE_KEY** — EC2 SSH private key for deployment access
- **STAGING_HOST** — Staging server IP address
- **STAGING_USER** — SSH username for staging server
- **PROD_HOST** — Production server IP address
- **PROD_USER** — SSH username for production server

> **Never commit secrets directly in code or workflow files.**
> Always use GitHub Secrets for sensitive information.

# Flask Jenkins CI/CD Pipeline

## Project Overview
This project demonstrates a Jenkins CI/CD pipeline that automates
building, testing, and deploying a simple Python Flask web application.

## Prerequisites
- Jenkins installed on AWS EC2 (Ubuntu 22.04)
- Python 3.x available on the Jenkins server
- GitHub account with webhook access
- Gmail account with App Password configured for notifications

## Project Structure
flask-jenkins-app/
├── app.py              # Main Flask application
├── test_app.py         # Unit tests (pytest)
├── requirements.txt    # Python dependencies
├── Jenkinsfile         # Jenkins pipeline definition
└── README.md           # This file


## Pipeline Stages
| Stage    | Description                                              |
|----------|----------------------------------------------------------|
| Checkout | Pulls the latest source code from GitHub                 |
| Build    | Creates a Python virtual environment, installs packages  |
| Test     | Runs pytest unit tests; pipeline fails if tests fail     |
| Deploy   | Deploys the Flask app to the staging server              |

## Setup Instructions

### 1. Jenkins Setup
1. Launch an AWS EC2 instance (Ubuntu 22.04, t3.micro)
2. Install Java 21 and Jenkins on the EC2 instance
3. Install plugins: GitHub Integration, Pipeline, Email Extension
4. Configure SMTP email (Gmail) under Manage Jenkins → System
5. Add GitHub credentials under Manage Jenkins → Credentials

### 2. Python Setup on Jenkins Server
```bash
sudo apt install -y python3 python3-pip python3.14-venv

### 3. Pipeline Job
    1. Create a new Pipeline job in Jenkins
    2. Point it to this repository and select "Pipeline script from SCM"
    3. Set branch to */main and Script Path to Jenkinsfile
    4. Enable "GitHub hook trigger for GITScm polling"

### 4. GitHub Webhook
    1. Go to Repository → Settings → Webhooks → Add webhook
    2. Set Payload URL to http://YOUR_JENKINS_IP:8080/github-webhook/
    3. Select content type: application/json
    4. Select "Just the push event"

### 5. Email Notifications
Configure Gmail SMTP in Jenkins:

Field	Value
SMTP Server	smtp.gmail.com
Username	your Gmail address
Password	Gmail App Password
Use SSL	Checked
SMTP Port	465

Running Locally

pip install -r requirements.txt
python app.py
# Visit http://localhost:5000

Running Tests Locally

pytest test_app.py -v

API Endpoints

Endpoint	Method	Description
/	GET	Returns welcome message
/health	GET	Health check

Notifications
The pipeline sends email notifications to the configured address on:

Build SUCCESS
Build FAILURE
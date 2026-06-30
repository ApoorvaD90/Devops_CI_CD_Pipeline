# Flask Jenkins CI/CD Pipeline
## Project Overview
This project demonstrates a Jenkins CI/CD pipeline that automates
building, testing, and deploying a simple Python Flask web application.
## Prerequisites
- Jenkins 2.x installed (with suggested plugins)
- Python 3.x available on the Jenkins server- GitHub account with webhook access
- Gmail account with App Password configured for notifications
## Project Structure
```
flask-jenkins-app/
├── app.py            
├── test_app.py       
# Main Flask application
# Unit tests (pytest)
├── requirements.txt  # Python dependencies
├── Jenkinsfile       
# Jenkins pipeline definition
└── README.md         
```
## Pipeline Stages
| Stage    
# This file
| Description                                              
|
|----------|----------------------------------------------------------|
| Checkout | Pulls the latest source code from GitHub                 
|
| Build    
| Test     
| Creates a Python virtual environment, installs packages  |
| Runs pytest unit tests; pipeline fails if tests fail     
|
| Deploy   | Deploys the Flask app to the staging server (main only)  |
## Setup Instructions
### 1. Jenkins Setup
1. Install Jenkins and start the service
2. Install plugins: GitHub Integration, Pipeline, Email Extension
3. Configure SMTP email (Gmail) under Manage Jenkins → System
4. Add GitHub credentials under Manage Jenkins → Credentials
### 2. Pipeline Job
1. Create a new Pipeline job in Jenkins
2. Point it to this repository and select "Pipeline script from SCM"
3. Set branch to `*/main` and Script Path to `Jenkinsfile`
4. Enable "GitHub hook trigger for GITScm polling"
### 3. GitHub Webhook
1. Go to Repository → Settings → Webhooks → Add webhook
2. Set Payload URL to `http://YOUR_JENKINS_IP:8080/github-webhook/`
3. Select content type: `application/json`
4. Select "Just the push event"
## Running Locally
```bash
pip install -r requirements.txt
python app.py
# Visit http://localhost:5000
```
## Running Tests Locally
```bash
pytest test_app.py -v
```
## API Endpoints
| Endpoint       
|----------------|--------|-------------------------------|
| /              
| /health        
| /add/{a}/{b}   | GET    
| Returns welcome message        
| GET    
| Health check endpoint          
| Returns sum of two numbers     
|
|
|
## Notifications
The pipeline sends email notifications to the configured address on:
- Build SUCCESS
- Build FAILURE
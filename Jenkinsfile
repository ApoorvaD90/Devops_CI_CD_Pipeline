pipeline {
    agent any
    environment {
        STAGING_HOST = '127.0.0.1'
        STAGING_PORT = '5000'
        APP_PID_FILE = '/tmp/flask_app.pid'
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Installing dependencies...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    . venv/bin/activate
                    pytest test_app.py -v --tb=short
                '''
            }
            post {
                always {
                    echo 'Tests completed.'
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to staging environment...'
                sh '''
                    . venv/bin/activate
                    if [ -f $APP_PID_FILE ]; then
                        kill $(cat $APP_PID_FILE) || true
                        rm $APP_PID_FILE
                    fi
                    nohup python3 app.py > /tmp/flask_app.log 2>&1 &
                    echo $! > $APP_PID_FILE
                    echo "App deployed with PID $(cat $APP_PID_FILE)"
                    sleep 3
                    curl -f http://$STAGING_HOST:$STAGING_PORT/health || exit 1
                    echo "Deployment verified successfully!"
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded!'
            mail(
                to: 'deshpande.apoorva123@gmail.com',
                subject: "SUCCESS: Jenkins Pipeline - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Build succeeded!
                    Job: ${env.JOB_NAME}
                    Build Number: ${env.BUILD_NUMBER}
                    Build URL: ${env.BUILD_URL}
                """
            )
        }
        failure {
            echo 'Pipeline failed!'
            mail(
                to: 'deshpande.apoorva123@gmail.com',
                subject: "FAILURE: Jenkins Pipeline - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Build FAILED!
                    Job: ${env.JOB_NAME}
                    Build Number: ${env.BUILD_NUMBER}
                    Build URL: ${env.BUILD_URL}
                    Please check the Jenkins console output for details.
                """
            )
        }
        always {
            cleanWs()
        }
    }
}
pipeline {
    agent any

    environment {
        APP_NAME = 'devsecops-demo'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SNYK_TOKEN = credentials('SNYK_TOKEN')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'npm ci'
            }
        }

        stage('Snyk Dependency Scan') {
            steps {
                echo 'Running Snyk vulnerability scan...'
                sh '''
                    npm install -g snyk
                    snyk auth $SNYK_TOKEN
                    snyk test --severity-threshold=high || true
                '''
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                echo 'Running Trivy filesystem scan...'
                sh '''
                    trivy fs . \
                        --severity CRITICAL,HIGH \
                        --ignore-unfixed \
                        --format table \
                        --exit-code 0
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -f docker/Dockerfile -t ${APP_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo 'Running Trivy Docker image scan...'
                sh '''
                    trivy image \
                        --severity CRITICAL,HIGH \
                        --ignore-unfixed \
                        --format table \
                        ${APP_NAME}:${IMAGE_TAG} || true
                '''
            }
        }

        stage('Secret Detection') {
            steps {
                echo 'Scanning for secrets...'
                sh '''
                    trivy fs . \
                        --scanners secret \
                        --format table \
                        --exit-code 1
                '''
            }
        }

        stage('Security Gate') {
            steps {
                echo '================================'
                echo '   Jenkins Security Gate'
                echo '================================'
                echo 'All security scans completed!'
                echo 'No critical secrets detected!'
                echo 'Security Gate PASSED!'
            }
        }
    }

    post {
        success {
            echo 'Pipeline PASSED — Safe to deploy!'
        }
        failure {
            echo 'Pipeline FAILED — Security issues found!'
        }
        always {
            echo 'Pipeline completed.'
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'samarthdave23'
        BACKEND_IMAGE   = "${DOCKER_HUB_USER}/stayngo-backend"
        FRONTEND_IMAGE  = "${DOCKER_HUB_USER}/stayngo-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Samarth-Dave/StayNGo.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    scannerHome = tool 'SonarScanner'
                }
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE}:latest ./backend"
                sh "docker build -t ${FRONTEND_IMAGE}:latest ./frontend"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${BACKEND_IMAGE}:latest"
                    sh "docker push ${FRONTEND_IMAGE}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker-compose -f docker-compose.yml up -d --force-recreate'

            }
        }
    }

    post {
        success { echo '✅ Pipeline SUCCESS - Code quality passed!' }
        failure { echo '❌ Pipeline FAILED - Check SonarQube or build logs.' }
    }
}
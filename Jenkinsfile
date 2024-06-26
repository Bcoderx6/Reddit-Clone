pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
        APP_NAME = "reddit-clone-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "mryash"
        DOCKER_CREDENTIALS_ID = 'DockerHub' // Update with your Docker Hub credentials ID
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Bcoderx6/Reddit-Clone'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-Clone-CI -Dsonar.projectKey=Reddit-Clone"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        docker.withRegistry('', DOCKER_PASS) {
                            def dockerImage = docker.build "${IMAGE_NAME}:${IMAGE_TAG}"
                            dockerImage.push()
                            dockerImage.push('latest')
                        }
                    }
                }
            }
        }
    }
}

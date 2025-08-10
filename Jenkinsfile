pipeline {
    agent any

    environment {
        DOCKER_REPO = "basilmohamed/java-app" 
    }

    tools {
        maven 'mvn-3-5-4'
        jdk 'java-17'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Fix mvnw Permissions') {
            steps {
                sh 'chmod +x mvnw'
            }
        }

        stage('Dependency Check') {
            steps {
                sh './mvnw dependency-check:check'
            }
        }

        stage('Build App') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Archive App') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh "echo ${DH_PASS} | docker login -u ${DH_USER} --password-stdin"
                    sh "docker build -t ${DOCKER_REPO}:latest ."
                    sh "docker push ${DOCKER_REPO}:latest"
                }
            }
        }
    }
}

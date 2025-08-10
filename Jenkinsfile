pipeline {
    agent { label "java" }

    tools {
        maven 'mvn-3-5-4'
        jdk 'java-11'
    }

    environment {
        DOCKER_REPO = "basil/java-app" // Docker Hub namespace/repo
    }

    stages {
        stage("Dependency Check") {
            steps {
                sh "./mvnw dependency-check:check"
                dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
        }

        stage("Build App") {
            steps {
                sh "./mvnw clean package -DskipTests"
            }
        }

        stage("Archive App") {
            steps {
                archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    sh "docker build -t ${DOCKER_REPO}:v${BUILD_NUMBER} ."
                    sh "docker tag ${DOCKER_REPO}:v${BUILD_NUMBER} ${DOCKER_REPO}:latest"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        sh "echo ${DH_PASS} | docker login -u ${DH_USER} --password-stdin"
                        sh "docker push ${DOCKER_REPO}:v${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_REPO}:latest"
                        sh "docker logout"
                    }
                }
            }
        }
    }
}

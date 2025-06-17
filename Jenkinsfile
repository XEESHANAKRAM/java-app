pipeline {
    agent any

    tools {
        jdk 'JDK'
        nodejs 'NodeJS'
    }

    parameters {
        string(name: 'ECR_REPO_NAME', defaultValue: 'java-app', description: 'Enter repository name')
    }

    environment {
        SCANNER_HOME = tool 'SonarQube-Scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/XEESHANAKRAM/java-app.git'
            }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Trivy File System Scan') {
    steps {
        sh '''#!/bin/bash
            trivy fs . > trivy.txt
        '''
    }
}

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'Docker') {
                        sh """
                            docker build -t ${params.ECR_REPO_NAME} .
                            docker tag ${params.ECR_REPO_NAME} xeeshanakram/${params.ECR_REPO_NAME}:latest
                            docker push xeeshanakram/${params.ECR_REPO_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Trivy') {
            steps {
                sh "trivy image xeeshanakram/java-app:latest > trivy.txt"
            }
        }

        
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}

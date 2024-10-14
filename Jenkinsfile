
pipeline {
    agent any
  
    
    environment {
        DOCKER_REGISTRY = 'docker.io' 
        DOCKER_REPOSITORY = 'arunagri03'
        IMAGE_NAME = 'jenkins'
        DOCKER_IMAGE_TAG = "${DOCKER_REGISTRY}/${DOCKER_REPOSITORY}/${IMAGE_NAME}:latest"
        DOCKER_PASSWORD = credentials('docker_token') 
    }

    tools {
        jdk 'JDK 11'
        maven 'Maven 3.8.1'  
    }

    stages {
        stage('Validating GitHub Access') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github_access', url: 'git@github.com:arun037/project-maven.git']])
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                sh '''
                    echo "$DOCKER_PASSWORD" | docker login ${DOCKER_REGISTRY} --username arunagri03 --password-stdin
                    docker build -t ${DOCKER_IMAGE_TAG} .
                    docker push ${DOCKER_IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withAWS(credentials: 'aws_id') {
                    sh 'aws eks update-kubeconfig --region ap-southeast-1 --name eks-cluster-1'
                    sh 'kubectl apply -f deploy-svc.yaml'
                }
            }
        }

        stage ('EKS Verification') {
            steps {
                withAWS(credentials: 'aws_id') {
                    sh 'aws eks update-kubeconfig --region ap-southeast-1 --name eks-cluster-1'  
                    sh 'kubectl get pods,deploy,svc -A'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment succeeded!'
        }
        failure {
            echo 'Build or deployment failed!'
        }
    }
}

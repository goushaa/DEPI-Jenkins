pipeline {
    agent any

    environment {
        AWS_ECR_REPO = 'kady-docker-repo'
        AWS_REGION = 'your-aws-region'
        DOCKER_IMAGE = "your-ecr-url/$AWS_ECR_REPO:latest"
        KUBECONFIG = '/path/to/your/kubeconfig'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/goushaa/DEPI-Code.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("$AWS_ECR_REPO:latest")
                }
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                script {
                    docker.withRegistry("https://<your-ecr-url>", 'aws-ecr-credentials') {
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f k8s-deployment.yaml --kubeconfig=$KUBECONFIG"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}

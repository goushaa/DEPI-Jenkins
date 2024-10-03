pipeline {
    agent any

    environment {
        AWS_ECR_REPO = 'kady-docker-repo'
        AWS_REGION = 'us-east-1'
        DOCKER_IMAGE = "522814709442.dkr.ecr.us-east-1.amazonaws.com/${AWS_ECR_REPO}"
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
                sh 'docker build -t dns-resolver .'
            }
        }

        stage('Debug Docker Images') {
            steps {
                // List Docker images
                sh 'docker images'
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    // Login to AWS ECR
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                        sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                script {
                    // Tag the image with the build number
                    def tags = ["${BUILD_NUMBER}", "latest"]
                    tags.each { tag ->
                        sh "docker tag dns-resolver:latest ${DOCKER_IMAGE}:${tag}"
                        sh "docker push ${DOCKER_IMAGE}:${tag}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Image pushed to ECR successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}

pipeline {
    agent any

    environment {
        AWS_ECR_REPO = 'kady-docker-repo'
        AWS_REGION = 'us-east-1'
        DOCKER_IMAGE = "522814709442.dkr.ecr.us-east-1.amazonaws.com/$AWS_ECR_REPO"
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
                sh 'docker build . -t dns-resolver'
            }
        }

        stage('Debug Docker Images') {
            steps {
                // Debugging step to list Docker images
                sh 'docker images'
            }
        }

       stage('Login to ECR') {
            steps {
                script {
                    // Login to AWS ECR
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                        // Use a variable for your ECR repository URI
                        def ecrUri = "${DOCKER_IMAGE}" // Make sure DOCKER_IMAGE is correctly set to your ECR URI
                        sh """
                        #!/bin/bash
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ecrUri}
                        """
                    }
                }
            }
        }


        stage('Tag & Push to ECR') {
            steps {
                script {
                    // Tag the image with the build number
                    sh "docker tag dns-resolver:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    // Tag the image as latest, overwriting the previous one
                    sh "docker tag dns-resolver:latest ${DOCKER_IMAGE}:latest"
                    // Push both tags to ECR
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
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

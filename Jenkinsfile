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

        stage('Tag & Push to ECR') {
            steps {
                script {
                    docker.withRegistry("https://522814709442.dkr.ecr.us-east-1.amazonaws.com", 'aws-ecr-credentials') {
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

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using the specified config file
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

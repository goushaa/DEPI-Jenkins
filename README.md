# Jenkins Pipeline Setup

## Manual Setup Steps

Before running the Jenkins pipeline, you need to complete the following manual setup steps on your EC2 instance where Jenkins is hosted:

1. **SSH into the `Jenkins` EC2 instance and set up the SSH key:**
```bash
ssh -i /path/to/your/key.pem ubuntu@your-ec2-public-ip
```

Manually create `k3sPair.pem` in the console and save it in a secure place
Then, add it to `Jenkins` instance
Inside the EC2 instance, run:
```bash
    sudo chmod 400 k3sPair.pem
    sudo chown jenkins:jenkins k3sPair.pem
    sudo mkdir -p /var/lib/jenkins/.ssh/
    sudo mv /home/ubuntu/k3sPair.pem /var/lib/jenkins/.ssh/
```


2. **Retrieve the Jenkins initial admin password:**
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

3. **Access Jenkins:**
   - Open your web browser and go to `http://your-ec2-public-ip:8080`.
   - Paste the initial admin password you retrieved in the previous step.
   - Follow the prompts to install the suggested plugins and create an admin user.
   - Once the setup is complete, log in with the admin user credentials you created.

4. **Install Required Plugins:**
   - Navigate to `Manage Jenkins -> Manage Plugins`.
   - Go to the `Available` tab and search for the following plugins:
     - **Amazon ECR**: This plugin enables Jenkins to interact with Amazon Elastic Container Registry (ECR).
     - **Docker Pipeline**: This plugin provides Docker support for Jenkins pipelines.
     - **GitHub Integration Plugin**: This plugin provides allows using Github Webhooks.
   - Select the plugins and click on the `Install without restart` button.

5. **Add AWS Credentials:**
   - Go to `Manage Jenkins -> Manage Credentials -> System -> Global`.
   - Click on `Add Credentials`.
   - From the `Kind` dropdown, select `AWS Credentials`.
   - Fill in your ECR access keys (Access Key ID and Secret Access Key) and provide an ID for the credentials.
   - Click `OK` to save.

6. **Create a New Pipeline Job:**
   - Go to the Jenkins dashboard.
   - Click on `New Item`.
   - Choose `CICD Pipeline` and give it a descriptive name (e.g., "Docker ECR Pipeline").
   - Select `Pipeline Script from SCM`.
   - For `SCM`, choose `Git`.
   - Enter the repository URL: `https://github.com/goushaa/DEPI-Jenkins.git`.
   - Set the branch to `*/main`.
   - Check the option of `GitHub hook trigger for GITScm polling`.
   - Click `Save` to create the job.

## Jenkinsfile Explanation

The provided `Jenkinsfile` defines a CI/CD pipeline for building a Docker image, pushing it to Amazon ECR, and deploying it to a target EC2 instance using Kubernetes. Here’s a breakdown of each stage:

### Pipeline Structure

- **Agent Declaration**: The pipeline can run on any available Jenkins agent.
- **Environment Variables**: 
  - `AWS_ECR_REPO`: The name of your ECR repository.
  - `AWS_REGION`: The AWS region where the ECR repository is located.
  - `DOCKER_IMAGE`: The full URL of the Docker image in ECR.
  - `TARGET_INSTANCE_NAME`: The name of the target EC2 instance where the application will be deployed.
  - `TARGET_KEY_PATH`: The path to the SSH key used to connect to the target instance.

### Stages

1. **Clone Repository**: 
   - Clones the specified Git repository containing the application code.

2. **Cleanup Docker Images**: 
   - Removes dangling images and old images that are not needed.

3. **Build Docker Image**: 
   - Builds a Docker image named `dns-resolver` using the Dockerfile in the repository.

4. **Debug Docker Images**: 
   - Lists the Docker images currently available on the Jenkins server for debugging purposes.

5. **Login to ECR**: 
   - Uses AWS CLI to authenticate Docker to your ECR repository using the AWS credentials stored in Jenkins.

6. **Tag & Push to ECR**: 
   - Tags the built Docker image with the current build number and `latest`, then pushes both tags to ECR.

7. **Fetch Target EC2 Details**: 
   - Retrieves the public IP address of the target EC2 instance and sets the SSH key path.

8. **Transfer Deployment File**: 
   - SSHs into the target EC2 instance to verify the connection, then securely copies the deployment YAML file to the instance.

9. **Apply Deployment**: 
   - SSHs into the target EC2 instance again and applies the Kubernetes deployment using the copied YAML file.

### Post Actions

- **Success**: Prints a success message if the pipeline completes without errors.
- **Failure**: Prints a failure message if any stage fails.


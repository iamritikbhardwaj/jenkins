# jenkins
Using Jenkins to automatically deploy your project involves setting up a **Continuous Integration/Continuous Deployment (CI/CD)** pipeline that automates the process of pulling your latest code changes, building Docker images, running tests, and deploying the project to your server.

Here's a step-by-step guide to set up Jenkins for automatic deployment:

### 1. **Install Jenkins**

First, you need to install Jenkins on your server or local machine. If you don't have Jenkins installed yet, follow the official installation guide: [Jenkins Installation](https://www.jenkins.io/doc/book/installing/).

If you want to run Jenkins in a Docker container, you can do this:

```bash
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins jenkins/jenkins:lts
```

### 2. **Install Necessary Plugins**
Once Jenkins is up and running, you'll need to install the following plugins to help automate deployments:

- **Git Plugin**: To pull code from your repository.
- **Docker Plugin**: If you're building and deploying Docker containers.
- **Pipeline Plugin**: To define the deployment pipeline as code (optional, but highly recommended).

To install plugins:
1. Navigate to **Manage Jenkins > Manage Plugins**.
2. Go to the **Available** tab, search for the necessary plugins (e.g., "Git", "Docker", "Pipeline"), and install them.

### 3. **Set Up a Jenkins Job**
You can set up Jenkins in two ways:
- **Freestyle Projects**: A simple way to configure Jenkins jobs using the web interface.
- **Pipeline Jobs**: A more flexible and powerful way to define the build and deployment process using code (recommended).

#### Option 1: **Freestyle Project (Simple Setup)**
This option is easier for beginners, but it has less flexibility compared to pipelines.

1. **Create a New Job**:
   - From the Jenkins dashboard, click **New Item**.
   - Name your job and select **Freestyle project**.
   - Click **OK**.

2. **Configure Source Code Repository**:
   - In the job configuration page, under **Source Code Management**, select **Git**.
   - Add the **repository URL** (e.g., GitHub, GitLab).
   - Add **credentials** (if needed) to access the repository.

3. **Add Build Steps**:
   - Under the **Build** section, you can add a step to run Docker commands or build your app.
   - For example, you can add a **Execute Shell** build step that runs the following commands to build and deploy your Docker container:
     ```bash
     docker build -t my-website .
     docker stop my-website-container || true
     docker rm my-website-container || true
     docker run -d -p 80:3000 --name my-website-container my-website
     ```

4. **Set Up Post-build Actions** (Optional):
   - You can configure Jenkins to notify you via email, Slack, or other methods upon success or failure.
   
5. **Save and Build**:
   - After saving the job configuration, click **Build Now** to test the deployment.
   
#### Option 2: **Pipeline Job (Recommended for Flexibility)**
Pipeline jobs are more powerful because they let you define the entire CI/CD process in code. You write a `Jenkinsfile` to describe the steps of the pipeline.

1. **Create a New Pipeline Job**:
   - From the Jenkins dashboard, click **New Item**.
   - Name your job and select **Pipeline**.
   - Click **OK**.

2. **Define Pipeline in Jenkinsfile**:
   - In the pipeline configuration page, under **Pipeline**, select **Pipeline script**.
   - In the script box, write your pipeline in **Groovy**.

Here’s an example `Jenkinsfile` for a Docker-based deployment:

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-website'
        DOCKER_CONTAINER = 'my-website-container'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the latest code from Git
                git 'https://github.com/your-repository.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image from the Dockerfile
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Stop & Remove Old Container') {
            steps {
                script {
                    // Stop and remove the old container if it exists
                    sh 'docker stop $DOCKER_CONTAINER || true'
                    sh 'docker rm $DOCKER_CONTAINER || true'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Run the new Docker container
                    sh 'docker run -d -p 80:3000 --name $DOCKER_CONTAINER $DOCKER_IMAGE'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
```

### 4. **Set Up Webhook (Optional but Recommended)**

To trigger the Jenkins job automatically when changes are pushed to your repository (for example, GitHub or GitLab), you can set up a **Webhook**. This will trigger Jenkins to start the job whenever new commits are pushed.

#### GitHub Webhook Setup:
1. Go to your repository’s settings on GitHub.
2. Under **Webhooks**, click **Add webhook**.
3. In the **Payload URL**, use your Jenkins URL (e.g., `http://your-jenkins-server.com/github-webhook/`).
4. Set the **Content type** to `application/json`.
5. Choose **Just the push event** to trigger on every push.
6. Click **Add webhook**.

Jenkins will now trigger the pipeline automatically when you push changes to the Git repository.

### 5. **Configure Notifications (Optional)**

To stay informed about the status of your Jenkins job, you can configure notifications to be sent on build success or failure.

- Go to the **Post-build Actions** section in Jenkins.
- You can configure notifications via:
  - **Email**: Notify a list of recipients on success or failure.
  - **Slack**: Send messages to a Slack channel using the **Slack Notification Plugin**.

### 6. **Monitor the Deployment**

Once everything is set up:
- **Git push**: Push changes to your repository.
- Jenkins will automatically:
  - Pull the latest code.
  - Build the Docker image.
  - Stop any old containers and start a new one.
  - Deploy the latest version of your app.

### Summary

By using Jenkins, you can automate the entire process of deploying your website with Docker:

1. Set up Jenkins and necessary plugins.
2. Create a Jenkins pipeline (either using a freestyle project or a `Jenkinsfile`).
3. Configure the pipeline to pull the latest code, build the Docker image, and deploy the container.
4. Set up webhooks to trigger Jenkins on code changes.
5. Configure notifications for build success or failure.

Once everything is set up, Jenkins will handle the rest, making deployments faster and more reliable!

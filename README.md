
# Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project!

## Phase 1: Initial Setup and Deployment
### Step 1: Launch EC2 (Ubuntu 22.04):

Provision an EC2 instance on AWS with Ubuntu 22.04.
Connect to the instance using SSH.





### Step 2: Clone the Code
Update all the packages and then clone the code.

Clone your application's code repository onto the EC2 instance:

```bash
  git clone https://github.com/saifsteyn/Netflix-Deploy.git
```



### Step 3: Install Docker and Run the App Using a Container:

Set up Docker on the EC2 instance:

```bash
  
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

Build and run your application using Docker containers:

```bash
  docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest

#to delete
docker stop <containerid>
docker rmi -f netflix
```
It will show an error cause you need API key


### Step 4: Get the API Key:

Open a web browser and navigate to TMDB (The Movie Database) website.
1. Click on "Login" and create an account.
2. Once logged in, go to your profile and select "Settings."
3. Click on "API" from the left-side panel.
4. Create a new API key by clicking "Create" and accepting the terms and conditions.
5. Provide the required basic details and click "Submit."
6. You will receive your TMDB API key.

Now recreate the Docker image with your api key:

```bash
  docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .

```

## Phase 2 : Security
### Install SonarQube and Trivy:

Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

```bash
 docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
To access:

publicIP:9000 (by default username & password is admin)

To install Trivy:
```bash
  sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy        
```
To scan image using Trivy

```bash
 trivy image <imageid>
```
## Integrate SonarQube and Configure:

Integrate SonarQube with your CI/CD pipeline.

Configure SonarQube to analyze code for quality and security issues.

```bash
  git clone https://github.com/N4si/DevSecOps-Project.git
```

## Phase 3: CI/CD Setup

### Install Jenkins for Automation:


### Update all the packages and then clone the code.

### Clone your application's code repository onto the EC2 instance:

Install Jenkins on the EC2 instance to automate deployment: Install Java

```bash
  sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)

#jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins in a web browser using the public IP of your EC2 instance.

   publicIp:8080

## Install Necessary Plugins in Jenkins:

#### Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 Eclipse Temurin Installer (Install without restart)

2 SonarQube Scanner (Install without restart)

3 NodeJs Plugin (Install Without restart)

4 Email Extension Plugin

### Configure Java and Nodejs in Global Tool Configuration
Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

### SonarQube
Create the token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

After adding sonar token

Click on Apply and Save

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

Create a Jenkins webhook

Configure CI/CD Pipeline in Jenkins:
Create a CI/CD pipeline in Jenkins to automate your application deployment.

```bash
  pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/saifsteyn/Netflix-Deploy.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
```
Certainly, here are the instructions without step numbers:

### Install Dependency-Check and Docker Tools in Jenkins

### Install Dependency-Check Plugin:

1. Go to "Dashboard" in your Jenkins web interface.
2. Navigate to "Manage Jenkins" → "Manage Plugins."
3. Click on the "Available" tab and search for "OWASP Dependency-Check."
4. Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.

### Configure Dependency-Check Tool:

After installing the Dependency-Check plugin, you need to configure the tool.
1. Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."
2. Find the section for "OWASP Dependency-Check."
3. Add the tool's name, e.g., "DP-Check."
4. Save your settings.

### Install Docker Tools and Docker Plugins:

1. Go to "Dashboard" in your Jenkins web interface.
2. Navigate to "Manage Jenkins" → "Manage Plugins."
3. Click on the "Available" tab and search for "Docker."
4. Check the following Docker-related plugins:

Docker

Docker Commons

Docker Pipeline

Docker API

docker-build-step

Click on the "Install without restart" button to install these plugins.

### Add DockerHub Credentials:

To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:

1. Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
2. Click on "System" and then "Global credentials (unrestricted)."
3. Click on "Add Credentials" on the left side.
4. Choose "Secret text" as the kind of credentials.
5. Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
6. Click "OK" to save your DockerHub credentials.

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.
```bash
  
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/saifsteyn/Netflix-Deploy.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourapikey> -t netflix ."
                       sh "docker tag netflix nasi101/netflix:latest "
                       sh "docker push nasi101/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nasi101/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
    }
}


If you get docker login failed errorr

sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins


```

## Create Kubernetes Cluster with Nodegroups

In this phase, you'll set up a Kubernetes cluster with node groups. This will provide a scalable environment to deploy and manage your applications.

### Monitor Kubernetes with Prometheus
Prometheus is a powerful monitoring and alerting toolkit, and you'll use it to monitor your Kubernetes cluster. Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.

### Install Node Exporter using Helm
To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

Add the Prometheus Community Helm repository:

Add the Prometheus Community Helm repository:

```bash
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
Create a Kubernetes namespace for the Node Exporter:

```bash
  kubectl create namespace prometheus-node-exporter
```
Install the Node Exporter using Helm:

```bash
  helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
```

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:

Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. You can do this by adding the following configuration to your prometheus.yml file:

```bash
    - job_name: 'Netflix'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['node1Ip:9100']
```
Replace 'your-job-name' with a descriptive name for your job. The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.

Don't forget to reload or restart Prometheus to apply these changes to your configuration.

To deploy an application with ArgoCD, you can follow these steps, which I'll outline in Markdown format:

### Access your Application

To Access the app make sure port 30007 is open in your security group and then open a new tab paste your NodeIP:30007, your app should be running.

### Phase 5: Cleanup

Cleanup AWS EC2 Instances:

Terminate AWS EC2 instances that are no longer needed.


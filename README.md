# Cloud-Engineering-Bootcamp

Steps to be followed as part of the bootcamp lab:


Continuous integration and Continuous Deployment Steps :
Step 1: Step up the environment.
1.	Launching the Server(Ubuntu Ec2 Instance) 
2.	Connect to the server.
3.	Install Jenkins.
Commands: 
1.	sudo apt update 
Install Jdk:
1.	sudo apt install openjdk-21-jdk -y
2.	sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

Install Jenkins:
3.	echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
4.	sudo apt update
5.	sudo apt install jenkins -y
6.	sudo systemctl status Jenkins
4.	Install Docker on Server:
Commands:
1.	sudo apt install gnupg2 pass -y
2.	sudo apt install docker.io -y
3.	sudo usermod -aG docker $USER
4.	newgrp docker
5.	sudo systemctl start docker
6.	sudo systemctl enable docker
7.	sudo systemctl status docker
8.	docker –version
9.	sudo usermod -a -G docker Jenkins
10.	sudo service jenkins restart
11.	sudo systemctl daemon-reload
12.	sudo service docker stop
13.	sudo service docker start
Install unzip:
14.	sudo apt install curl unzip
15.	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
16.	unzip awscliv2.zip
17.	sudo ./aws/install

5.	Setup your Jenkins by reseting the password and by installing all the packages.
Jenkins Plugins to install:
-	Open Jenkins.
-	Go to "Manage Jenkins" > "Manage Plugins".
-	Install the following plugins: 
1.	Docker Pipeline
2.	Pipeline: AWS Steps
3.	Amazon ECR
6.	Create an AWS ECR Repository. 
7.	Create a Jenkins Pipeline Job:
7.1.	 Open Jenkins.
7.2.	 Click on "New Item".
7.3.	 Enter a name for the job (e.g., BuildAndPushDockerImage).
7.4.	 Select "Pipeline" and click "OK".
7.5.	 Configure the Pipeline:
7.6.	 Scroll down to the "Pipeline" section.
7.7.	 Select "Pipeline script" and enter the pipeline script.
8.	Configure the ECS.
1.	Create Cluster.
1.1.	Go to ECS > Clusters > Create Cluster
1.2.	Choose: Networking only (Fargate)
1.3.	Name: flask-cluster

2.	Create ECS IAM Execution Role:
2.1.	Go to IAM > Roles > Create Role:
2.2.	Trusted entity: Elastic Container Service > ECS task
2.3.	Attach policy: AmazonECSTaskExecutionRolePolicy
2.4.	Name: ecsTaskExecutionRole

3.	Create Task definition:
3.1.	Go to ECS > Task Definitions > Create new Task Definition
3.2.	Launch type: FARGATE
3.3.	 Task role: ecsTaskExecutionRole
3.4.	 Task memory: 512 MiB (or more)
3.5.	Task CPU: 256 (or more)
3.6.	Add container:
3.7.	Container name: flask-app
3.8.	Image: 256707027315.dkr.ecr.eu-north-1.amazonaws.com/flask-hello-world:latest
3.9.	Port mappings: container port 5000 <your application port>
3.10.	Create

4.	Create Target group
4.1.	Go to EC2 > Target Groups > Create Target Group
4.2.	 Choose:
4.2.1.	Target type: IP
4.2.2.	Protocol: HTTP
4.2.3.	Port: 5000 (the port on which your container/application is running)
4.2.4.	VPC: Choose your VPC
4.3.	Name it e.g., flask-target-group
4.4.	Health check path: /health (from your Flask app)
4.5.	Create

5.	Create Application Load Balancer (ALB)
5.1.	Go to EC2 > Load Balancers > Create Load Balancer
5.2.	 Choose Application Load Balancer
5.3.	 Name: e.g., flask-alb
5.4.	 Scheme: Internet-facing
5.5.	 Listeners: HTTP (port 80)
5.6.	AZs: Select public subnets
5.7.	Security group: Allow inbound on port 80
5.8.	Target group: Select the one created earlier (flask-target-group)
5.9.	Finish creating the ALB

6.	Create Service:
6.1.	 Go to ECS > Clusters > flask-cluster > Services > Create
6.2.	Launch type: FARGATE
6.3.	Task definition: Select the one created
6.4.	Service name: flask-service
6.5.	Number of tasks: 1 (or more)
Networking:
6.6.	 VPC: Select one with public subnets
6.7.	Subnets: Select public subnets
6.8.	Security group: Allow inbound HTTP (port 5000 internally, or 80 externally via ALB)
6.9.	Load balancer: Application Load Balancer
6.10.	Listener: HTTP:80
6.11.	Target group: Select flask-target-group

ECS Security Group :
Inbound: 
Type	Protocol	Port	Source
Custom TCP	TCP	5000	Security group of the ALB ✅

Outbound:
Type	Protocol	Port	Destination
All traffic	All	All	0.0.0.0/0

How to set up the Access Key and Secret Keys in Jenkins:
set the aws access key n secret key into Jenkins -> Manage Jenkins > System > Environment Variables
•	AWS_ACCESS_KEY_ID=your_access_key
•	AWS_SECRET_ACCESS_KEY=your_secret_key
•	AWS_DEFAULT_REGION=your_region

Attaching the Sample Jenkinsfile:

pipeline {
  agent any 
  environment{
      registry = "<Account-ID>.dkr.ecr.<Region>>.amazonaws.com/flask-hello-world"
      }
  stages {
    stage('checkout') {
      steps {
        checkout scmGit(branches: [[name:'*/main']], extensions: [], userRemoteConfigs:[[credentialsId:'github-cred',
        url: 'https://github.com/panwarsakshi241/Cloud-Engineering-Bootcamp.git']])
      }
    }
    // Building Docker images
    stage('Building image') {
      steps {
        script { dockerImage = docker.build registry }
      }
    }
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
      steps {
        script {
          sh 'aws ecr get-login-password --region <region> | docker login --username  AWS --password-stdin <Account-ID>.dkr.ecr.<Region>.amazonaws.com' 
          sh 'docker push <Account-ID>.dkr.ecr.<Region>.amazonaws.com/flask-hello-world:latest'
        }
      }
    }
    stage('Docker Run') {
      steps {
        script {
          sh '''
            docker rm -f myflask-container || true
            docker run -d -p 8096:5000 --rm --name myflask-container <Account-ID>.dkr.ecr.<Region>.amazonaws.com/flask-hello-world:latest
        '''
        }
      }
    }
  }
}



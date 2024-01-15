---
title: "Mastering DevOps on AWS"
datePublished: Mon Jan 15 2024 17:36:48 GMT+0000 (Coordinated Universal Time)
cuid: clrf7j2ip000908l943wlbfjd
slug: mastering-devops-on-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705314247549/c3f50dfa-6cac-4a30-800e-b56eb3aa85c0.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1705340187948/93edbbf8-929f-4d80-92b1-dfc2bca39a76.jpeg
tags: docker, github, devops, ecs, trainwithshubham, aws-ecr

---

In this blog, we'll delve into the seamless integration of DevOps practices on the AWS cloud, harnessing the power of GitHub, EC2, ECR, Docker, ECS, and Cloud Watch. Our journey will culminate in the deployment of a Node.js application onto ECS, showcasing the efficient orchestration of these tools in a cohesive development and deployment pipeline. Let's explore each step of this process to empower your team with an effective and streamlined DevOps workflow.

Without further delay, let's dive into the practical aspects of our DevOps journey on AWS.

### 1- **First Create an EC2 instance on aws:**

* **Access AWS Console:** Log in to your AWS account and go to the AWS Management Console.
    
* **Navigate to EC2 Dashboard:** Find and select "EC2" under the Compute section in the Services drop down.
    
* **Launch an Instance:** Click "Instances" in the left sidebar and press "Launch Instance."
    
* **Choose AMI:** Select an Amazon Machine Image (AMI) that suits your requirements.
    
* **Choose Instance Type:** Pick an instance type based on your application's needs.
    
* **Configure Instance:** Set instance details, including network settings and IAM role if necessary.
    
* **Add Storage:** Specify the size of the root volume for your instance.
    
* **Configure Security Group:** Define security settings, opening ports as needed for your application.
    
* **Review and Launch:** Review your configuration and click "Launch."
    
* **Create Key Pair:** Choose an existing key pair or create a new one for secure instance access.
    
* **Launch Instances:** Click "Launch Instances" to provision your EC2 instance.
    
* **Access EC2 Instance using EC2 Instance Connect:**
    
    1. In the AWS Management Console, navigate to the "Instances" section under EC2.
        
    2. Select the EC2 instance you created.
        
    3. Click the "Connect" button.
        
    4. Choose "EC2 Instance Connect" as the connection method.
        
    5. Click "Connect" to access your instance securely.
        

### 2- Clone the GitHub repository with source code:

* Clone the specified GitHub repository onto your instance
    
    Github link : [https://github.com/shubzz-t/node-todo-cicd.git](https://github.com/shubzz-t/node-todo-cicd.git)
    
    ```bash
    #Command to clone the source code to your instance
    git clone https://github.com/shubzz-t/node-todo-cicd.git
    ```
    

### 3- Install the AWS CLI and Docker onto EC2:

* Install aws cli
    
    ```bash
    #Update the ec2 packages
    sudo apt-get update
    
    #Install aws cli
    sudo apt-get install -y awscli
    
    #Verify aws cli installation
    aws --version
    ```
    
    Install Docker:
    
    ```bash
    #Command to install docker
    sudo apt-get install -y docker.io
    
    #Start docker service
    sudo systemctl start docker
    
    #Add user to the docker group
    sudo usermod -aG docker $USER
    
    #Verify docker installation
    docker --version
    ```
    

### 4 - Creating repository and pushing it onto ECR (Elastic Container Registry):

* **Creating Repository:**
    
    * In the AWS Management Console, go to the "Services" drop down and select "ECR" under the "Container" section.
        
    * Click on the "Create repository" button.
        
    * Enter a unique repository name.
        
    * In content types select operating systems and architectures.
        
    * Click on the "Create repository" button.
        
* **Authenticate Docker to ECR:**
    
    * In the ECR repository, click on the "View push commands" button.
        
    * Copy and paste the `docker login` command into your EC2 instance terminal to authenticate Docker to your ECR registry.
        
    * You need to create the IAM user with cli access for the secret key and password.
        
* **Build, Tag and Push Docker Image:**
    
    1. **Navigate to Your Project Directory:**
        
        * Move to the directory where your Dockerfile and application code are located.
            
    2. **Build , Tag and Push Docker Image:**
        
        * Build your Docker image using the following command from the "View push commands"
            
        * Tag the Docker image with the ECR repository URI.
            
        * Push the Docker image to your ECR repository.
            
    3. **Verify Image in ECR:**
        
        * Go back to the ECR console and refresh the page.
            
        * You will see the pushed Docker image in the repository.
            

### 5- Creating ECS cluster and task definition:

To create an Amazon ECS cluster with the necessary configuration, including a task definition and service, follow these steps:

1. **Navigate to ECS console:**
    
    * In the AWS Management Console, go to the "Services" dropdown and select "ECS" under the "Compute" section.
        
2. **Create an ECS cluster:**
    
    * Click on "Clusters" in the left sidebar. Click the "Create Cluster" button.
        
    * Choose a cluster template based on your needs. For example, you can start with the "EC2 Linux + Networking" template.
        
    * Configure the cluster settings:
        
        * **Cluster Name:** Enter a unique name for your ECS cluster.
            
        * **Infrastructure type:** AWS Fargate.
            
        * **Monitoring :** Use container insights.
            
3. **Create a Task Definition:**
    
    * 1. In the ECS console, navigate to "Task Definitions" in the left sidebar.
            
        2. Click the "Create new Task Definition" button.
            
        3. Select the launch type compatibility (Fargate).
            
        4. Configure your task definition:
            
            * **Task Definition Name:** Enter a name for your task definition.
                
            * **Task Role:** Choose the IAM role that provides permissions to your containers.
                
            * **Task Execution Role:** Choose the IAM role for your task execution role.
                
            * Define container definitions, image URL ( image path from the ECR) , port mappings(map port 80 with 8000), environment variables, etc.
                
4. **Running the task in cluster:**
    
    * Now run the task then select the cluster name in which we want to run the task and click on run.
        
    * The task will be running and you will be able to see the status.
        
5. **Access the application on IP and Port:**
    
    * After the task is running then from the configuration get the public IP of the container and access the application on publicIP:8000 (8000 is the mapped port).
        
    * If not able to access the the application then from ENI id go to the security group settings and then change the inbound rules for the mapped port.
        
    * Also you will be able to see the logs onto the cloud watch log group.
        

Fantastic! Congratulations on completing your DevOps journey using ECS, ECR, and deploying a Node.js application. Feel free to share any specific content you'd like assistance with, such as the blog's conclusion or any specific points you want to highlight in the final paragraphs!!
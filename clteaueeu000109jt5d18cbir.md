---
title: "Jenkins CI/CD Simplified"
datePublished: Tue Mar 05 2024 11:41:14 GMT+0000 (Coordinated Universal Time)
cuid: clteaueeu000109jt5d18cbir
slug: jenkins-cicd-simplified
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1709638778325/291ba095-74c6-48cc-bbe0-5b1b063af5e8.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1709638867351/91920dbb-3d02-4b8b-8f5b-101c448665bd.png
tags: aws, github, sonarqube, kubernetes, maven, devops, jenkins, minikube, argocd

---

  
In this project, we've upgraded our DevOps game by using a mix of powerful tools. First, we set up an AWS EC2 instance and installed Jenkins, SonarQube, and Docker, making it a one-stop shop for our CI/CD needs. Jenkins became our main player, smoothly connecting with DockerHub and SonarQube for seamless development and testing.

Our process starts with fetching code from GitHub and following the steps in the Jenkinsfile for deployment. We also brought in SonarQube to double-check our code quality before moving forward. Once everything's good, Jenkins builds our Docker image and sends it off to DockerHub.

Then comes ArgoCD, our deployment superhero. It grabs the image and setup files from DockerHub and handles deployment on our Minikube cluster. With ArgoCD in charge, we're confident our deployments in Kubernetes are consistent and reliable across different setups.

Let's automate, test, build, and deploy with precision and efficiency! 🔥

### Tools Used:

* AWS(EC2)
    
* Github
    
* Maven
    
* Jenkins
    
* Docker
    
* SonarQube
    
* ArgoCD
    
* Kubernetes(Minikube)
    

To access the code for this project and start revolutionizing your DevOps practices, clone the GitHub repository from the following link: [https://github.com/shubzz-t/java-maven-sonar-argocd-k8s.git](https://github.com/shubzz-t/java-maven-sonar-argocd-k8s.git).

### 1\. Creating the EC2:

* **Access AWS Console:** Log in to your AWS account and go to the AWS Management Console.
    
* **Navigate to EC2 Dashboard:** Find and select "EC2" under the Compute section in the Services drop down.
    
* **Launch an Instance:** Click "Instances" in the left sidebar and press "Launch Instance."
    
* **Choose AMI:** Select an Amazon Machine Image (AMI) that suits your requirements. Choose the ubuntu t2.large instance.
    
* **Choose Instance Type:** Pick an instance type based on your application's needs.
    
* **Configure Instance:** Set instance details, including network settings and IAM role if necessary.
    
* **Add Storage:** Specify the size of the root volume for your instance.
    
* **Configure Security Group:** Define security settings, open all the ports for practice purpose.
    
    **Review and Launch:** Review your configuration and click "Launch."
    
* **Create Key Pair:** Choose an existing key pair or create a new one for secure instance access.
    
* **Launch Instances:** Click "Launch Instances" to provision your EC2 instance.
    
* **Access EC2 Instance using EC2 Instance Connect:**
    
    1. In the AWS Management Console, navigate to the "Instances" section under EC2.
        
    2. Select the EC2 instance you created.
        
    3. Click the "Connect" button.
        
    4. Choose "EC2 Instance Connect" as the connection method.
        
    5. Click "Connect" to access your instance securely.
        

### 2- Installing Jenkins, Jenkins Plugins:

* **Firstly to to install Jenkins we require Java use the below steps to install java :**
    
    ```bash
    #Update the packages
    sudo apt update
    #Install the openjdk 17
    sudo apt install fontconfig openjdk-17-jre
    #Check if java is installed
    java -version
    ```
    
* **Installing Jenkins:**
    
    ```bash
    #Copy the jenkins key and packeges
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
      https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    
    #Update the packeges
    sudo apt-get update
    
    #Install Jenkins
    sudo apt-get install jenkins
    
    #Enable Jenkins
    sudo systemctl enable jenkins
    
    #Start Jenkins
    sudo systemctl start jenkins
    ```
    

### 3- Installing Plugins, Docker and SonarQube:

* **Install Docker on Ubuntu server:**
    
    ```bash
    #Installing docker
    sudo apt-get install docker.io
    
    #Add user ubuntu and jenkins to docker group
    sudo usermod -aG docker $USER
    sudo usermod -aG docker jenkins
    ```
    
* **Install SonarQube on Ubuntu Server:**
    
    ```bash
    #Install unzip software
    apt install unzip
    
    #Add sonarqube user
    adduser sonarqube
    
    #Change user to sonarqube
    sudo su - sonarqube
    
    #Download the sonarqube packages
    wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
    
    #Unizip the packeges
    unzip *
    
    #Change permissions
    chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
    
    #Change owner to sonarqube
    chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
    
    #Go inside the sonarqube directory
    cd sonarqube-9.4.0.54424/bin/linux-x86-64/
    
    #Run the sonarqube executable
    ./sonar.sh start
    #SonarQube will start on the port 9000 by default can access on 
    # publicip:9000 username:admin password:admin as default
    ```
    
* **Install the required Jenkins plugins:**
    
    In Jenkins go to &gt; Manage Jenkins &gt; Install Plugins &gt; Search and install the below plugins.
    
    1. Docker Plugin
        
    2. Docker Pipeline Plugin
        
    3. Sonar Scanner Plugin
        

### 4- Configure Github, DockerHub and SonarQube credentials in Jenkins:

Inside Jenkins go to Jenkins &gt; Manage Jenkins &gt; Manage Credentials &gt; System &gt; Global Credentials &gt; Add credentials &gt;

* **Add Github Credentials:**
    
    1. Select the secret text option.
        
    2. Insert id as github.
        
    3. Insert GitHub token under secret for accessing the repository.
        
    4. Click on save.
        
* **Add DockerHub Credentials:**
    
    1. Select the Username and Password option.
        
    2. Insert id as dock-cred.
        
    3. Insert username of your dockerhub account.
        
    4. Insert password for your dockerhub account.
        
    5. Click save.
        
* **Add SonarQube credentials:**
    
    1. Select secret text option.
        
    2. Insert id as sonar.
        
    3. Insert the secret token generated from SonarQube administrator dashboard.
        
    4. Click save.
        

### 5- Installing Minikube and configuring ArgoCD:

We will be installing the Minikube cluster on personal computer or other EC2 instance as this EC2 instance might get slow due to huge applications running on it.

* Installing Minikube:
    
    ```bash
    #Install docker
    sudo apt install -y docker.io
    
    #Add user to the docker
    sudo usermod -aG docker $USER
    
    #Get the minikube packages
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    
    #Install Minikube from package
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    
    #Check version
    minikube version
    
    #Get kubectl package
    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
    
    #Change permission
    chmod +x kubectl
    
    #Move kubectl to bin for allover access
    sudo mv kubectl /usr/local/bin/
    
    #Start Minikube with docker driver
    minikube start --driver=docker
    
    #Verify Minikube status
    minikube status
    ```
    
* Install ArgoCD operator on Minikube
    
    You can follow the steps mentioned on OperatorsHub.io &gt; ArgoCD &gt; Install for installing ArgoCD on Kubernetes. If not below are the steps:
    
    ```bash
    #Install Operator Lifecycle Manager (OLM)
    curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.27.0/install.sh | bash -s v0.27.0Install Operator Lifecycle Manager (OLM)
    
    # Install operator i.e. argocd
    kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
    
    # Check operator comes up or not
    kubectl get csv -n operators
    ```
    
* Bring ArgoCD in action:
    
    1. Create a yml file with following script:
        
        argo\_basic.yml
        
        ```yaml
        apiVersion: argoproj.io/v1alpha1
        kind: ArgoCD
        metadata:
          name: example-argocd
        spec: {}
        ```
        
    2. Run the above script using the below command:
        
        ```bash
        kubectl apply -f argo_basic.yml
        ```
        
    3. Change service type of example-argocd-server service file to access outside cluster:
        
        * Command to get the service file and edit:
            
            ```bash
            #List all the Minikube service files
            kubectl get svc
            
            # example-argocd-server is the argocd service file edit it's service to Nodeport from clusterIP
            kubectl edit svc example-argocd-server
            ```
            
    4. Get the password for the ArgoCD:
        
        * Password is present inside the secrets manifest/
            
        * Command to get the secret file :
            
            ```bash
            #List all the secrets
            kubectl get secret
            
            # example-argocd-cluster is the secret file where password is present copy that
            ```
            
    5. Decrypt the password:
        
        * On terminal use the following command to decrypt password.
            
            ```bash
            echo 'copied password' | base64 -d
            ```
            
    6. Get the URL to access the ArgoCD:
        
        * Command to get the URL:
            
            ```bash
            minikube service list
            #Copy the url in front of the example-argocd-server 
            ```
            
    7. Start ArgoCD:
        
        * Paste the copied URL in browser and hit enter.
            
        * Enter the username : admin and password is which you decrypted.
            
        * And you are in ArgoCD dashboard.
            

### 6- Creating Jenkins Pipeline:

* Go to Jenkins and create new item &gt; Name the project &gt; Click Pipeline &gt; Ok.
    
* Scroll down to pipeline section.
    
* Select Pipeline script from SCM option from drop down.
    
* Select SCM as git.
    
* Enter the repository URL.
    
* Enter the branch of your git main/master.
    
* Script path i.e. path to your Jenkinsfile here in my case it is spring-boot-app/JenkinsFile.
    
* Click Apply and Save.
    

### 7- Using ArgoCD to deploy Application:

* Click New App
    
* Enter application name &gt; Project name &gt; Sync policy = Automatic &gt; Repository URL = Your\_repo\_url .
    
* Path i.e. the path to you deployment file in this case it is spring-boot-app-manifests/deployment.yml.
    
* Cluster URL keep it as the default cluster.
    
* Namespace keep it as the default Namespace.
    
* Click on create.
    

Boom! Your application is now live on the Minikube cluster, courtesy of ArgoCD. Congratulations on successfully deploying your Spring Boot application!

### **Thanks for Joining Us:**

A huge thank you for joining us on this DevOps journey! If you have questions, thoughts, or want to share your experiences, we'd love to hear from you. Happy coding, and may your apps always scale to new heights!
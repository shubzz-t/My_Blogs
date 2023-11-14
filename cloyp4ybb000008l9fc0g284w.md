---
title: "Navigating the DevOps Landscape: Building a Java CI/CD Pipeline"
seoTitle: "CICD pipeline for deploying java application"
seoDescription: "Step-by-step guide to creating a CI/CD pipeline for a Java application. We'll break down the logical workflow of a DevOps project, showing you the ins and o"
datePublished: Tue Nov 14 2023 18:58:13 GMT+0000 (Coordinated Universal Time)
cuid: cloyp4ybb000008l9fc0g284w
slug: navigating-the-devops-landscape-building-a-java-cicd-pipeline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699988109829/970c5482-2c36-48ab-9318-1d059c2587fc.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1699988127218/757ab7cb-6f5c-4230-b88a-d36529e39258.png
tags: ansible, kubernetes, maven, jenkins, ci-cd

---

Get ready for an exciting journey into the world of DevOps! Whether you're a seasoned pro with Git, Jenkins, Maven, Ansible, Docker, and Kubernetes, or you're just starting out, our blog is designed to help you understand how these tools work together.

We've put together a step-by-step guide to creating a CI/CD pipeline for a Java application. We'll break down the logical workflow of a DevOps project, showing you the ins and outs of GitHub, Jenkins, Maven, Docker, Ansible, and Kubernetes, and how they all fit into a real-world project.

By the end of this program, our aim is to boost your confidence so you can set up your own CI/CD pipeline. As you read through, soak in the practical knowledge shared, and get ready to bring these DevOps practices to life in your own projects. Let's dive into this journey together and take your expertise in CI/CD and DevOps to the next level.

Before we building a Java CI/CD pipeline, feel free to explore the complete source code and project files in our GitHub repository. You can find everything you need for reference and follow along with the tutorial.

Github Link :- [https://github.com/shubzz-t/hello\_world\_java](https://github.com/shubzz-t/hello_world_java)

### *Prerequisite:*

*AWS Account*

Github Account

*DockerHub Account*

*Amazon Linux 2 Instance :-*

Before we dive into building our CI/CD pipeline, let's ensure we have a running Amazon Linux 2 - 3 instances with all traffic enabled. This instances will serve as our playground for learning and experimenting with the DevOps tools.

1. **Sign in to the AWS Management Console:** Go to [AWS Console](https://aws.amazon.com/) and sign in to your AWS account.
    
    In the AWS Management Console, navigate to the EC2 service.
    
2. **Launch an Instance:**
    
    * Click on the "Instances" link on the left-hand side. Click the "Launch Instance" button.
        
3. **Choose an Amazon Machine Image (AMI):**
    
    * In the "Choose an Amazon Machine Image (AMI)" step, select "Amazon Linux 2 AMI."
        
4. **Choose an Instance Type:**
    
    * In the "Choose an Instance Type" step, select an instance type based on your requirements. The default t2.micro instance is eligible for the AWS Free Tier.
        
5. **Configure Instance:**
    
    * In the "Configure Instance" step, leave the default settings or configure the instance details as needed.
        
6. **Add Storage:**
    
    * In the "Add Storage" step, configure the storage settings as needed.
        
7. **Add Tags:**
    
    * In the "Add Tags" step, add any tags if required (optional).
        
8. **Configure Security Group:**
    
    * In the "Configure Security Group" step, create a new security group or choose an existing one.
        
    * For learning purposes, you can create a new security group allowing all traffic. To do this, add a rule with "Type" set to "All traffic" and "Source" set to "Anywhere" (0.0.0.0/0).
        
9. Left hand side enter the number of instances as and click on Launch Instances.
    
10. **Rename Instances:**
    
    Instance 1 = Jenkins\_Server
    
    Instance 2 = Ansible\_Server
    
    Instance 3 = Kubernetes\_Server
    

Now, your Amazon Linux 2 - 3 instances are being launched with all traffic allowed. Once it's running, you can connect to it using SSH with the private key you downloaded.

Remember that keeping all traffic open is not recommended for production environments. For production, you would typically want to restrict access to specific ports based on your application's needs.

## *Step 1 : Setting up the Jenkins\_Server*

**1) Installing JDK and Jenkins Integration Tool:**

* As a root use command:
    
    ```bash
    #REPOSITORY REDHAT PACKAGES FOR JENKINS
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    
    #You can install java 11 using 
    sudo amazon-linux-extras install java-openjdk11
    
    #You can install using
    yum install jenkins
    
    # You can enable the Jenkins service to start at boot with the command:
    systemctl enable jenkins
    
    # You can start the Jenkins service with the command:
    systemctl start jenkins
    
    # You can check the status of the Jenkins service using the command:
    systemctl status jenkins
    ```
    

1. **Install Maven :**
    

* The steps to install Apache Maven on your Amazon Linux 2 instance:
    
    ```bash
    #Switch to /opt directory
    cd /opt
    
    #Download the maven binaries from official maven site (you can download latest whichever available)
    wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
    
    #Extract the downoladed binaries
    tar -xvzf apache-maven-3.9.5-bin.tar.gz
    ```
    

**3)** **Install Git :**

* ```bash
    #Installing the git
    sudo yum install git
    ```
    

**4)** **Configure Java and Maven Environment Variables:**

Let's set up the environment variables for Java and Maven. These variables ensure that our CI/CD pipeline can smoothly build and deploy Java applications.

* We need to set the environment variables inside the ~/.bashrc file.
    
* JAVA\_HOME for Java and M2, M2\_HOME for Maven
    
    ```bash
    #Open the .bashrc file with the VI editor
    vi ~/.bashrc
    ```
    
* You can find the Java path using the "whereis java" command and copy-paste it in .bashrc :
    

> > JAVA\_HOME='/usr/lib/jvm/&lt;your java path&gt;'
> > 
> > M2\_HOME='/opt/maven'
> > 
> > M2='/opt/maven/bin'
> > 
> > PATH=$JAVA\_HOME/bin:$PATH:$M2\_HOME:$M2
> > 
> > export PATH

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699755028550/25ce9d00-fb61-445f-b8b8-3a097cefbf94.png align="center")

* Now execute the .bashrc file so that its get's executed and path's get loaded.
    
    ```bash
    #Executee the modified .bashrc file
    source ~/.bashrc
    
    #Check paths are visible on the $PATH variable
    echo $PATH
    ```
    
    With Maven installed and environment variables set, you've now set up Jenkins, JDK, and Maven on your Amazon Linux 2 instance. These tools are essential components for building and automating your Java applications in a CI/CD pipeline.
    

**5)** **Access Jenkins Web Interface and Install Plugins**

* **Access Jenkins Web Interface:** Open a web browser and navigate to [`http://your_server_ip:8080`](http://your_server_ip:8080)
    
* **Enter Initial Administrator Password:**
    
    * Retrieve the initial administrator password:
        
        ```bash
        sudo cat /var/lib/jenkins/secrets/initialAdminPassword
        ```
        
    * Copy the password from the terminal and paste it into the Jenkins web interface.
        
* **Complete Jenkins Setup:** Follow the on-screen instructions to complete the Jenkins setup, including installing suggested plugins.
    
* **Create Jenkins Admin User:** Create an admin user with the required credentials.
    
    Now, Jenkins is ready for action! You've successfully configured Jenkins on your server, and it's set up with the necessary plugins for your CI/CD workflow.
    

**6)** **Configure Tools in Jenkins**

* **Navigate to "Manage Jenkins":**
    
    * Click on "Manage Jenkins" in the left-hand sidebar.
        
* **Configure JDK:**
    
    * Click on "Global Tool Configuration."
        
    * Find the "JDK" section and click "JDK installations"
        
    * Click "Add JDK" and provide a name.
        
    * Specify the path to your JDK installation in the "JAVA\_HOME" field.
        
* **Configure Maven:**
    
    * Find the "Maven" section and click "Maven installations"
        
    * Click "Add Maven" and provide a name.
        
    * Specify the path to your Maven installation in the "MAVEN\_HOME" field.
        
* **Configure Git:**
    
    * Find the "Git" section.
        
    * Click "Git installations"
        
    * Click "Add Git" and provide a name.
        
    * Specify the path to your Git executable in the "Path to Git executable" field.
        
    * Click "Save."
        

## *Step 2 : Setting up the Ansible\_Server*

**1) Install Ansible**:

* ```bash
            #Install ansible 
            sudo amazon-linux-extras install ansible2 -y
    ```
    

**2) Create and configure ansible user:**

* Create user "**ansadmin**". To add the ansadmin user to the **sudoers** file (typically managed by the `visudo` command) to grant them sudo privileges, you can follow these steps:
    
    ```bash
    #Command to open the sudoers file
    sudo visudo
    
    #Add the below line to the opened file
    ansadmin ALL=(ALL) NOPASSWD: ALL
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699800381798/bfa05f97-03fa-4416-8c36-8c95f80be31e.png align="center")
    
* Next uncomment the **PasswordAuthentication yes** line in the **/etc/ssh/sshd\_config** file and comment the **PasswordAuthentication no** line:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699800571073/4baae2e5-f4c3-4b0e-9894-2453587a51c5.png align="center")
    
* After making changes to the SSH configuration file, it's a good practice to reload the SSH daemon to apply the changes. Here's the command to do that:
    
    ```bash
    #Command to reload the sshd service
    sudo systemctl restart sshd.service
    ```
    

**3) Generating and copying the ssh keys:**

* Generate ssh keys for ansible as well as kubernetes server using the command:
    
    ```bash
    #Command to generate the ssh keys
    ssh-keygen
    ```
    
* Use the ssh-copy-id command to copy your public key to the ansadmin as well as kubernetes\_server root user's **authorized\_keys** file on the remote server:
    
    ```bash
    #Command to copy public key to ansible server ansadmin
    ssh-copy-id <your-ansible-server-ip>
    
    #Command to copy public key to kubernetes_server root user
    ssh-copy-id root@<your-kubernetes-server-ip>
    ```
    
* Configuring the ansible inventory/hosts file:
    
* Open the ansible inventory file which is present at **/etc/ansible/hosts** location:
    
    ```bash
    #Opening the inventory file for editing
    sudo vi /etc/ansible/hosts
    ```
    
* Add the ansible\_server IP to the above hosts file under the group(anygroup you can create) here \[ansible\] and \[kuberentes\] is group and down are the servers IP :
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699812430034/2d14fa77-c5da-4a6b-ad76-ff882d85ff8f.png align="center")
    

**4) Install Docker :**

* To install Docker on your system, you can use the following command.
    
    ```bash
    #Command to install docker
    sudo yum install docker
    ```
    
* After installing Docker, you need to enable and start the Docker service. On Linux systems that use systemd (like Amazon Linux 2), you can use the following commands:
    
    ```bash
    #Command to enable the docker service
    sudo systemctl enable docker
    
    #Command to start the docker service
    sudo systemctl start docker
    
    #Command to check status of the docker service
    sudo systemctl status docker
    ```
    
* Now we need to add the ansadmin user to the docker group, allowing them to run Docker commands without using sudo, you can use the following command:
    
    ```bash
    #Command to add the ansadmin to the docker group
    sudo usermod -aG docker ansadmin
    ```
    
* Now create the working directory in /opt/ directory with docker as name(it can have any name here I used docker) for keeping the playbooks and docker files with ansadmin permissions:
    
    ```bash
    #Changing the group and user permissions for docker file
    sudo chgrp ansadmin:ansadmin docker
    ```
    

**5) Creating Dockerfile:**

* Go inside the /opt/docker directory and create the **Dockerfile**.
    
    ```bash
    cd /opt/docker/
    vi Dockerfile
    ```
    
* Copy the below code into your created Dockerfile.
    
    ```go
    FROM tomcat:latest
    RUN cp -R  /usr/local/tomcat/webapps.dist/*  /usr/local/tomcat/webapps
    COPY ./*.war /usr/local/tomcat/webapps
    ```
    
* > Dockerfile : builds the tomcat image and copies the webapps.dist files and directories to webapps to run the tomcat default files. Then copy command is used to copy the .war file from current location to containers specified location.
    

**6) Creating playbook to build and push image:**

* Create a playbook to build the image from the Dockerfile and push that image to the dockerhub.
    
* Here I will create a playbook named build\_push\_img.yml in the /opt/docker/ directory.
    
    ```go
    vi build_push_img.yml
    ```
    
* Copy the below code to the opened build\_push\_img.yml
    
    ```yaml
    ---
    - hosts: ansible
      
      tasks:
        - name: create docker image
          command: docker build -t regapp:latest .
          args: 
           chdir: /opt/docker
    
        - name: Create tag to push image to docekrhub
          command: docker tag regapp:latest shubzz/java_app_img:latest
    
        - name: Push the image to dockerhub
          command: docker push shubzz/java_app_img:latest
    ```
    
    Ansible file explanation:
    
    > **ansible** = group specified in inventory file
    > 
    > **command : docker build -t regapp:latest .**
    > 
    > command to build image from current directory Dockerfile with regapp:latest tag.
    > 
    > **command: docker tag regapp:latest shubzz/java\_app\_img:latest** Command to tag the image here shubzz is my dockerhub username , java\_app\_img is my dockerhub repository and latest is the tag.
    > 
    > **command: docker push shubzz/java\_app\_img:latest**
    > 
    > command to push the docker image to the dockerhub repository .
    

**7) Configuring the Ansible server details in the Jenkins "Configure System":**

* Firstly install the "Publish over ssh" plugin from "Manage Jenkins" -&gt; Plugins -&gt; Available Plugins -&gt; Search and install "Publish over ssh"
    
* **Navigate to "Manage Jenkins" &gt; "Configure System":**
    
    * Click on "Manage Jenkins" in the left-hand sidebar.
        
    * Scroll down and click on "Configure System."
        
* **Find the "Publish over ssh" Section**
    
* **Enter Ansible Server Details:**
    
    * Enter the following details:
        
        **Name:** A logical name for your Ansible configuration (e.g., "MyAnsibleServer").
        
        **Hostname:** The private IP address or hostname of your Ansible server.
        
        **Username:** The username to use for connecting to the Ansible server in our case it's ansadmin.
        
* **Select "Use password authentication, or use a different key" from Advanced Section:**
    
    * Enter ansadmin user password in Password section.
        
* **Save Changes and Test Connection**
    

## *Step 3 : Setting up the Jenkins\_Server*

**1) Install AWS CLI:**

* Follow the below commands to install the AWS CLI:
    
    ```bash
    #Download the AWSCLI package from the official aws site
    curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
    
    #Unzip the downloaded package
    unzip awscliv2.zip
    
    #Install the cli 
    sudo ./aws/install
    ```
    

**2) Install kubectl**

* Follow the below commands for kubectl installation:
    
    ```bash
    #Download the KUBECTL package from the official aws site
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.7/2023-11-02/bin/linux/amd64/kubectl
    
    # Make the kubectl binary executable
    chmod +x kubectl
    
    # Move the binary to a directory in your PATH
    sudo mv kubectl /usr/local/bin/
    
    #Verify the installation
    kubectl --version
    ```
    

**3) Install eksctl:**

* Follow the below commands for eksctl installation:
    
    ```bash
    # for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
    ARCH=amd64
    PLATFORM=$(uname -s)_$ARCH
    
    #Download the EKSCTL package from the official aws site
    curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
    
    #Extract eksctl package and copy to /tmp and remove tar file
    tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
    
    # Move the eksctl binary to a directory in your PATH from tmp
    sudo mv /tmp/eksctl /usr/local/bin
    
    # Verify the installation
    eksctl version
    ```
    
    > Resolving kubectl and Kubernetes cluster version mismatches is a common challenge. Ensure compatibility by updating kubectl to match the cluster version using kubectl version --client, preventing issues and enhancing management efficiency
    

**4) Create an IAM Role with AdministratorAccess:**

* Open the AWS Management Console and navigate to the IAM service.
    
* In the left navigation pane, click on "Roles" and then "Create role."
    
* Choose "AWS service" as the trusted entity type, select "EC2" from the use case options, and click "Next: Permissions."
    
* In the "Filter policies" search bar, type and select "AdministratorAccess," then click "Next: Tags."
    
* (Optional) Add any tags you want for the role, then click "Next: Review."
    
* Enter a meaningful name for the role and provide a description. Click "Create role."
    

**5) Create Deployment and Service file:**

* Creating app-deployment.yml file and adding the below code in it.
    
    ```bash
    #Creating the app-deployment.yml file and add the below code to it.
    vi app-deployment.yml
    ```
    
    ```yaml
    apiVersion: apps/v1 
    kind: Deployment
    metadata:
      name: app-dep
      labels: 
         app: regapp
    
    spec:
      replicas: 2 
      selector:
        matchLabels:
          app: regapp
    
      template:
        metadata:
          labels:
            app: regapp
        spec:
          containers:
          - name: regapp
            image: shubzz/java_app_img
            imagePullPolicy: Always
            ports:
            - containerPort: 8080
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
    ```
    
* Creating app-service.yml file and copying the blow code to it.
    
    ```bash
    #Creating the app-service.yml file and add the below code to it.
    vi app-service.yml
    ```
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: app-svc
      labels:
        app: regapp 
    spec:
      selector:
        app: regapp 
    
      ports:
        - port: 8080
          targetPort: 8080
    
      type: LoadBalancer
    ```
    

**6) Create a kubernetes cluster:**

* Create a kubernetes cluster where we can run our application on multiple servers here is how we can create cluster:
    
    ```bash
    #command to create cluster regionname eg:ap-south-1
    eksctl create cluster --name clustername  --region regionname  --node-type t2.small
    ```
    

**7) Create playbook on ansible\_server to run regapp-deploy and regapp-service yml files inside /opt/docker/:**

* Create playbook deployment\_service.yml
    
    ```yaml
    ---
    - hosts: kubernetes
      #become: true
      user: root
    
      tasks:
              - name: Deploy app in k8s using deployment
                command: kubectl apply -f app-deployment.yml
              - name: deploy app on k8s service
                command: kubectl apply -f app-service.yml
              - name: update deployment with new pods if image updated in docker hub
                command: kubectl rollout restart deployment.apps/regapp
    ```
    

## *Step 4 : Creating Jenkins CD Job*

* Open Jenkins and follow the below steps:-
    
    **1) Create a New Jenkins Job:**
    
    * Click on "New Item" to create a new Jenkins job.
        
    
    **2) Choose Freestyle Project:**
    
    * Select "Freestyle project" as the project type.
        
    
    **3) Configure General Settings:**
    
    * Provide a meaningful name for your project.
        
    * Set any necessary project description.
        
    
    **4) Configure Post-Build Actions:**
    
    * Scroll down to the "Post-build Actions" section.
        
    * Click on "Add post-build action" and select "Send build artifacts over SSH."
        
    
    **5) Select our ansible server:**
    
    * From drop down select our MyAnsibleServer which we have provided while server configuration.
        
    
    **6) Configure Execution on Ansible Server:**
    
    * In the "Exec command" section, add the command to execute your Ansible playbook.
        
        ```bash
        #Command with which ansible will execute the playbook
        ansible-playbook /opt/docker/build_push_img.yml
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699950611288/3c695176-5c1b-4d86-9c8e-371eeeb0d917.png align="center")
        
    
    **7) Save the Jenkins Job:**
    
    * Save your Jenkins job configuration.
        

## *Step 5 : Creating Jenkins CI Job*

* Creating Jenkins CI job for that login to Jenkins and follow the steps:
    
    **1) Create a New Jenkins Job:**
    
    * Click on "New Item" to create a new Jenkins job.
        
    
    **2) Choose Maven Project:**
    
    * Select "Maven project" as the project type.
        
    
    **3) Configure General Settings:**
    
    * Provide a meaningful name and description for your project.
        
    
    **4) Configure Source Code Management:**
    
    * Select "GitHub" as the source code management system.
        
    * Add your GitHub repository URL.
        
    * Specify the branch as "master."
        
    
    **5) Configure Build Triggers:**
    
    * Under "Build Triggers," select "Poll SCM."
        
    * Schedule it as `* * * * *` to check for changes in the GitHub repository every minute.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699949584259/76f89ee4-ab3a-4cf3-8b73-704943a698fe.png align="center")
        
    
    **6) Configure Prebuilt Steps:**
    
    * Set the root POM to "pom.xml."
        
    * Under "Goals and options," add "clean install."
        
    
    **7) Configure "Send build artifacts over SSH" Section:**
    
    * Add the name of the SSH server.
        
    * In the "Transfer Set" section:
        
        * Under "Source files," add `webapp/target/*.war` to specify the WAR file to be copied.
            
        * Under "Remove prefix," add `webapp/target/` to avoid copying the whole path.
            
    * Under "Remote directory," add the path where you want to copy the WAR file on the Ansible server our case it is `//opt//docker`.
        
    
    **8) Configure Exec Command:**
    
    * In the "Exec command" section, add the command to execute your Ansible playbook.
        
        * Example command: ansible-playbook /opt/docker/deployment\_service.yml;
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699950108622/65996334-79a6-4d99-bcff-72764e2b271f.png align="center")
            
    
    **9) Configure Post-Build Actions:**
    
    * Scroll down to the "Post-build Actions" section.
        
    * Click on "Add post-build action" and select "Build other projects."
        
    
    **10) Specify the CD Job:**
    
    * In the "Projects to build" section, add the name of your Continuous Delivery (CD) Jenkins job that you built previously.
        
    * Check the option "Trigger only if build is stable."
        
    
    **11) Specify the CD Job:**
    
    * In the "Projects to build" section, add the name of your Continuous Delivery (CD) Jenkins job that you built previously.
        
    * Check the option "Trigger only if build is stable."
        
    
    **12) Save and Run the Jenkins Job:**
    
    * Save your Jenkins job configuration.
        
    * Run the job to trigger the Maven build and deployment process.
        
    
    Adjust the details(file names , server names , software versions , job names , dockerhub repository name) based on your specific environment and requirements.
    
    **Here we have completed our CICD pipeline successfully....**
    

In this guide, we've outlined key steps to craft a robust CI/CD pipeline for Java applications. Explore GitHub for version control, Jenkins for CI, Maven for automation, Docker for containerization, Ansible for deployment, and Kubernetes for orchestration. Our step-by-step instructions offer insights for seamless tool integration, providing a comprehensive view of the DevOps lifecycle. With this knowledge, you can efficiently navigate modern software development complexities, ensuring automation and scalability. Remember, continuous improvement is paramount, and adapting these practices to your project will empower success. For any queries, feel free to reach out. Thank you for your valuable time.
---
title: "Zero Downtime Deployment: Blue-Green Strategy for Java MySQL Applications on EKS"
datePublished: Sat Oct 05 2024 11:28:51 GMT+0000 (Coordinated Universal Time)
cuid: cm1w2mrfc00080amfbk3333ib
slug: zero-downtime-deployment-blue-green-strategy-for-java-mysql-applications-on-eks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1728127358558/d9b73208-0420-4afd-8275-dcfc44ad8055.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1728127721254/a836fb02-89ab-4edc-84fd-178f351303e0.png
tags: docker, aws, sonarqube, kubernetes, devops, terraform, jenkins, cicd-cjy1vtdk2005kjjs17n8couc3, eks, bluegreen-deployment

---

In today's fast-paced development environment, ensuring application availability and minimizing downtime during deployment is crucial. One of the most effective deployment strategies to achieve this is **blue-green deployment**, which allows for seamless updates and faster rollbacks in case of issues.

In this blog, I will guide you through the process of deploying a **Java-based MySQL application** using **blue-green deployment** to achieve zero downtime and rapid rollback. We'll leverage a **CI/CD pipeline** integrated with tools like **Terraform** for provisioning an **Amazon EKS** cluster, **Git**, **Maven**, **SonarQube**, **Trivy**, **Nexus**, **Docker**, and **Kubernetes** to automate the entire deployment process.

**What is Blue-Green Deployment?**

**Blue-green deployment** is a deployment strategy designed to reduce downtime and minimize the risks involved in releasing new software versions. In this approach, two identical environments are maintained: **Blue** (the currently live environment) and **Green** (the environment with the new application version).

Here’s how it works:

1. **Blue Environment (Current Production)**: The "blue" environment runs the live version of the application.
    
2. **Green Environment (New Version)**: The new version of the application is deployed to the "green" environment, which is identical to the blue environment.
    
3. **Switching Traffic**: Once the green environment is fully tested and validated, traffic is seamlessly switched from the blue to the green environment. This ensures that users experience no downtime.
    
4. **Rollback**: In case of any issues with the green environment, traffic can easily be switched back to the blue environment, providing a fast and reliable rollback option.
    

By the end of this tutorial, you'll have a fully automated pipeline that ensures smooth application updates with zero downtime, high availability, and robust security checks.

Prerequisites:

* AWS Account (with access key and secret access key)
    

### 1\. Provision and Configure EKS :

* Create EC2 ubuntu and ssh into the instance:
    
    1. Update the instance:
        
        ```plaintext
        sudo apt update
        ```
        
    2. Install AWS CLI:
        
        ```plaintext
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
        ```
        
    3. Configure:
        
        ```plaintext
        aws configure
        ```
        
        Provide the access key and secret access key for the aws user and enter.
        
    4. Install terraform:
        
        ```plaintext
        sudo snap install terraform --classic
        ```
        
    5. Clone the git repository for the EKS cluster code which you can get on:
        
        [https://github.com/shubzz-t/Blue-Green-Deployment-CICD.git](https://github.com/shubzz-t/Blue-Green-Deployment-CICD.git)
        
    6. Provision Infrastructure using terraform go into the folder /cluster:
        
        ```plaintext
        #Terraform initialization command
        terraform init
        
        #Terrafrom plan for planning resources
        terraform plan
        
        #Terraform apply command to apply changes to AWS
        terraform apply --auto-approve
        ```
        
    7. Connecting to the cluster:
        
        ```plaintext
        aws eks --region ap-south-1 update-kubeconfig --name cluster-name
        ```
        
    8. To access the EKS cluster using the Jenkins pipeline we need Jenkins to have access to EKS, for that we will be creating the Service Account in webapps namespace which jenkins will use to access the EKS resources.
        
        * Creating namespace webapps in EKS:
            
            ```plaintext
              kubectl create ns webapps
            ```
            
        * Creating service account first create service\_acc.yml with content as:
            
            ```plaintext
              apiVersion: v1
              kind: ServiceAccount
              metadata:
                name: jenkins
                namespace: webapps
            ```
            
        * To create service account run command:
            
            ```plaintext
              kubectl apply -f service_acc.yml
            ```
            
        * Now we will be creating role to attach with the service account we will create a file role.yml with content:
            
            ```plaintext
              apiVersion: rbac.authorization.k8s.io/v1
              kind: Role
              metadata:
                name: app-role
                namespace: webapps
              rules:
                - apiGroups:
                      - ""
                      - apps
                      - autoscaling
                      - batch
                      - extensions
                      - policy
                      - rbac.authorization.k8s.io
                  resources:
                    - pods
                    - secrets
                    - componentstatuses
                    - configmaps
                    - daemonsets
                    - deployments
                    - events
                    - endpoints
                    - horizontalpodautoscalers
                    - ingress
                    - jobs
                    - limitranges
                    - namespaces
                    - nodes
                    - pods
                    - persistentvolumes
                    - persistentvolumeclaims
                    - resourcequotas
                    - replicasets
                    - replicationcontrollers
                    - serviceaccounts
                    - services
                  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
            ```
            
        * To create role run command:
            
            ```plaintext
              kubectl apply -f role.yml
            ```
            
        * Next step is to assign the created role to the service account i.e. binding the role to service account for that create file bind\_role.yml with content:
            
            ```plaintext
              apiVersion: rbac.authorization.k8s.io/v1
              kind: RoleBinding
              metadata:
                name: app-rolebinding
                namespace: webapps 
              roleRef:
                apiGroup: rbac.authorization.k8s.io
                kind: Role
                name: app-role 
              subjects:
              - namespace: webapps 
                kind: ServiceAccount
                name: jenkins
            ```
            
        * To bind role with service account run command:
            
            ```plaintext
              kubectl apply -f bind_role.yml
            ```
            
        * Next create secret token for the jenkins service account, will create file jenkins\_token.yml with content:
            
            ```plaintext
              apiVersion: v1
              kind: Secret
              type: kubernetes.io/service-account-token
              metadata:
                name: mysecretname
                annotations:
                  kubernetes.io/service-account.name: jenkins
            ```
            
        * To create secret in webapps namespace run command:
            
            ```plaintext
              kubectl apply -f jenkins_token.yml -n webapps
            ```
            
        * We will be using this secret for jenkins to EKS communication, to get the token run command:
            
            ```plaintext
              kubectl describe secret mysecretname -n webapps
            ```
            
        * You will get the secret copy that secret and paste it to the Jenkins Secret.
            

### 2\. Jenkins, SonarQube and Nexus:

Create 3 t2.medium ubuntu EC2 instances with 25 GB Storage.Name them as Jenkins, SonarQube and Nexus.

* **Configure Jenkins Instance:**
    
    1. SSH into the jenkins instance.
        
    2. Install Java for Jenkins:
        
        ```plaintext
        # Install OpenJDK 17 JRE Headless
        sudo apt install openjdk-17-jre-headless -y
        ```
        
    3. Install Jenkins:
        
        ```plaintext
        # Download Jenkins GPG key
        sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
          https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
        
        # Add Jenkins repository to package manager sources
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
          https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
          /etc/apt/sources.list.d/jenkins.list > /dev/null
        
        # Update package manager repositories
        sudo apt-get update
        
        # Install Jenkins
        sudo apt-get install jenkins -y
        
        #Enable jenkins
        sudo systemctl enable jenkins
        ```
        
    4. Access the jenkins server on &lt;ec2-public-ip&gt;:8080 and provide username as admin and password we will get using the command:
        
        ```plaintext
          sudo cat /var/lib/jenkins/secrets/initialAdminPassword
        ```
        
    5. Install plugins in jenkins by clicking Dashboard &gt; Manage Jenkins &gt; Plugins:
        
        \- SonarQube scanner
        
        \- Config File Provider
        
        \- Maven Integration
        
        \- Pipeline Maven Integration
        
        \- Kubernetes
        
        \- Kubernetes Credentials
        
        \- Kubernetes CLI
        
        \- Kubernetes Client API
        
        \- Docker
        
        \- Docker Pipeline
        
        \- Pipeline stage view
        
    6. **Installing Tools:**
        
        In Jenkins &gt; Tools
        
        1. Maven: name= maven3; Install from apache; version=3.9.8
            
        2. SonarQube Scanner: name=sonar-scanner; Install from maven central; version=6.1.0.4477
            
    7. Install docker on Jenkins server using:
        
        ```plaintext
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc
        
        # Add the repository to Apt sources:
        echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
          $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
          sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
        
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        
        sudo chmod 666 /var/run/docker.sock
        ```
        
    8. Install Trivy:
        
        ```plaintext
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy
        ```
        
    9. Install kubectl on Jenkins instance:
        
        ```plaintext
          sudo snap install kubectl --classic
        ```
        
* **Configure SonarQube Server:**
    
    1. SSH into the sonarqube instance.
        
    2. Install docker on the instance:
        
        ```plaintext
        sudo apt install docker.io
        ```
        
    3. Run the sonarqube container:
        
        ```plaintext
        sudo docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
        ```
        
    4. You can access the sonarqube using the &lt;sonar-ec2-public-ip&gt;:9000.
        
    5. Next to sign in into sonarqube enter username: admin and password: admin.
        
    6. Create token by clicking Administration &gt; Security &gt; Users &gt; Generate token &gt; copy token.
        
* **Configure Nexus Server:**
    
    1. SSH into the nexus instance.
        
    2. Install docker on the instance:
        
        ```plaintext
        sudo apt install docker.io
        ```
        
    3. Run Nexus container:
        
        ```plaintext
        deploymentReposudo docker run -d --name nexus -p 8081:8081 sonatype/nexus3:latest
        ```
        
    4. You can access the nexus using the &lt;nexus-ec2-public-ip&gt;:8081.
        
    5. Next to you need to sign in into nexus using the username: admin and password we need to check for the file /nexus-data/admin.password inside the nexus container for that follow below commands:
        
        ```plaintext
          sudo docker exec -it containerid /bin/bash
          cd sonatype-work   
          cd nexus3 
          cat admin.password
        ```
        

### 3\. Adding Credentials in Jenkins:

To store credentials inside Jenkins, go to Jenkins &gt; Manage Jenkins &gt; Manage Credentials &gt; System &gt; Global Credentials &gt; Add credentials &gt;

1. Git Credentials: Username & Password (github username:token)
    
2. DockerHub: Username with password ( dockerhub username:password ).
    
3. SonarQube: Secret Text ( Copied SonarQube token).
    
4. Kubernetes: Secret Text ( jenkins service account token).
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728124516483/7ca0771e-aaf5-4488-8d66-6dd30e42cbbc.png align="center")
    

### 4\. Adding Sonar Scanner in Jenkins:

* To configure sonarqube server go to Jenkins server &gt; Dashboard &gt; Manage Jenkins &gt; System
    
* Under SonarQube server click on add SonarQube
    
* Add name= sonar-server
    
* url= &lt;sonarserver-ip&gt;:9000
    
* Select created credential as token. Click Apply.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724538858402/3276f833-c492-4969-94aa-691facc3adb9.png?auto=compress,format&format=webp align="left")
    

### 5\. Configure Nexus:

* **Add URL to pom.xml**
    
    1. To configure nexus first go to nexus &gt; browse &gt; copy maven releases url and paste it in the &lt;maven-release&gt; url section inside pom.xml of source code.
        
    2. For maven-snapshot copy the url from nexus &gt; browse &gt; copy maven-snapshot url and paste in the &lt;maven-snapshots&gt; url section inside pom.xml of source code.
        
        ```plaintext
         <distributionManagement>
                 <repository>
                     <id>maven-releases</id>
                     <url>http://3.110.172.144:8081/repository/maven-releases/</url>
                 </repository>
                 <snapshotRepository>
                     <id>maven-snapshots</id>
                     <url>http://3.110.172.144:8081/repository/maven-snapshots/</url>
                 </snapshotRepository>
             </distributionManagement>
        ```
        
* **Nexus Credentials:**
    
    1. Go to Jenkins &gt; Dashboard &gt; Manage Jenkins &gt; Managed Files &gt; Click Add new config &gt; Id= maven-settings &gt; Next
        
    2. You will get the maven settings file, inside file find the &lt;server&gt; &lt;/server&gt; section and uncomment it. And make 2 copies of it.
        
        ```plaintext
         <server>
             <id>maven-releases</id>
             <username>nexus_username</username>
             <password>nexus_password</password>
         </server>
         <server>
             <id>maven-snapshots</id>
             <username>nexus_username</username>
             <password>nexus_password</password>
         </server>
        ```
        
    3. Click Save.
        

### 6\. Creating Jenkins Pipeline:

* Click Dashboard &gt; New Item &gt; Project name &gt; select pipeline &gt; OK
    
* Click Discard old builds &gt; Max build =2
    
* Next create pipeline:
    
    ```plaintext
    pipeline {
        agent any
        
        tools{
            maven 'maven3'
        }
        
        parameters {
            choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
            choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
            booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
        }
        
        environment {
            IMAGE_NAME = "shubzz/bankapp"
            TAG = "${params.DOCKER_TAG}"  // The image tag now comes from the parameter
            KUBE_NAMESPACE = 'webapps'
            SCANNER_HOME= tool 'sonar-scanner'
    
        }
    
        stages {
            stage('Git Checkout') {
                steps {
                    git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/shubzz-t/Blue-Green-Deployment-CICD.git'
                }
            }
            
            stage('Compile'){
                steps{
                    sh "mvn compile"
                }
            }
            
            stage('Tests'){
                steps{
                    sh "mvn test -DskipTests=true"
                }
            }
            
            stage("Trivy FS scan"){
                steps{
                    sh "trivy fs --format table -o fs.html ."
                }
            }
            
            stage('SonarQube Analysis') {
                steps {
                    withSonarQubeEnv('sonar') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=multitier -Dsonar.projectName=multitier -Dsonar.java.binaries=target" 
                    }
                }
            }
            
            stage("Sonar Quality Gate Check"){
                steps{
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: false
                    }
                }
            }
            
            stage("Build"){
                steps{
                    sh "mvn package -DskipTests=true"
                }
            }
            
            stage('Trivy FS Scan') {
                steps {
                    sh "trivy fs --format table -o fs.html ."
                }
            }
            
            stage("Pulish to Nexus"){
                steps{
                    withMaven(globalMavenSettingsConfig: 'mvn-global', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                        sh 'mvn deploy -DskipTests=true'
                    }
                }
            }
            
            stage('Docker build') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                        }
                    }
                }
            }
            
            stage('Trivy Image Scan') {
                steps {
                    sh "trivy image --format table -o image.html ${IMAGE_NAME}:${TAG}"
                }
            }
            
            stage('Docker Push Image') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker push ${IMAGE_NAME}:${TAG}"
                        }
                    }
                }
            }
            stage('Deploy MySQL Deployment and Service') {
                steps {
                    script {
                        withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CAFA65A9A51DA473D77884FE6006404A.gr7.ap-south-1.eks.amazonaws.com') {
                            sh "kubectl apply -f mysql-ds.yml -n ${KUBE_NAMESPACE}"  // Ensure you have the MySQL deployment YAML ready
                        }
                    }
                }
            }
            
            stage('Deploy SVC-APP') {
                steps {
                    script {
                        withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CAFA65A9A51DA473D77884FE6006404A.gr7.ap-south-1.eks.amazonaws.com') {
                            sh """ if ! kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}; then
                                    kubectl apply -f bankapp-service.yml -n ${KUBE_NAMESPACE}
                                  fi
                            """
                       }
                    }
                }
            }
            
            stage('Deploy to Kubernetes') {
                steps {
                    script {
                        def deploymentFile = ""
                        if (params.DEPLOY_ENV == 'blue') {
                            deploymentFile = 'app-deployment-blue.yml'
                        } else {
                            deploymentFile = 'app-deployment-green.yml'
                        }
    
                        withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CAFA65A9A51DA473D77884FE6006404A.gr7.ap-south-1.eks.amazonaws.com') {
                            sh "kubectl apply -f ${deploymentFile} -n ${KUBE_NAMESPACE}"
                        }
                    }
                }
            }
            
            stage('Switch Traffic Between Blue & Green Environment') {
                when {
                    expression { return params.SWITCH_TRAFFIC }
                }
                steps {
                    script {
                        def newEnv = params.DEPLOY_ENV
    
                        // Always switch traffic based on DEPLOY_ENV
                        withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CAFA65A9A51DA473D77884FE6006404A.gr7.ap-south-1.eks.amazonaws.com') {
                            sh '''
                                kubectl patch service bankapp-service -p "{\\"spec\\": {\\"selector\\": {\\"app\\": \\"bankapp\\", \\"version\\": \\"''' + newEnv + '''\\"}}}" -n ${KUBE_NAMESPACE}
                            '''
                        }
                        echo "Traffic has been switched to the ${newEnv} environment."
                    }
                }
            }
            
            stage('Verify Deployment') {
                steps {
                    script {
                        def verifyEnv = params.DEPLOY_ENV
                        withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CAFA65A9A51DA473D77884FE6006404A.gr7.ap-south-1.eks.amazonaws.com') {
                            sh """
                            kubectl get pods -l version=${verifyEnv} -n ${KUBE_NAMESPACE}
                            kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}
                            """
                        }
                    }
                }
            }
            
        }
    }
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728124894003/3acdc827-93ff-421e-80cf-7c476d01e0df.png align="center")
    

### 7\. Getting Domain Ready

This step is optional step if you don't have any domain name registered you can skip this step.

* Go to your domain provider in my case it is [**GoDaddy.com**](http://godaddy.com/).
    
    * Click on DNS and you will get all the records in that click on the CNAME record.
        
    * Add your load balancer dns so that all request coming to your domain will be routed to load balancer dns.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724495007319/5212315f-5bb7-47e2-acf4-9b7add2e4402.png?auto=compress,format&format=webp align="left")
        
    * Click Save and it will take time to reflect.
        
    * Blue Deployment with Bank name as Goldencat.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728125440521/975e6af8-07b4-4aaf-aea6-f52951762ea8.png align="center")
        
    * New green deployment with traffic switch to green deployment and changed bank name.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728125581506/a5cc9909-8729-43dd-90e7-27d3b79b45a9.png align="center")
        

**Conclusion:**

By implementing the blue-green deployment strategy, we’ve achieved a robust and flexible approach for deploying Java MySQL applications on AWS EKS with zero downtime and faster rollback capabilities. Leveraging tools like Terraform for infrastructure, and integrating Git, Maven, SonarQube, Trivy, Nexus, and Docker in the CI/CD pipeline ensures that each release is not only automated but also secure and reliable. This setup empowers teams to continuously deploy with confidence, reducing risks while maintaining seamless application performance.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
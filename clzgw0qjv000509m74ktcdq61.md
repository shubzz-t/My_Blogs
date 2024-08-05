---
title: "Building a Robust CI/CD Pipeline with GitHub Actions for Multi-Cluster Deployment"
datePublished: Mon Aug 05 2024 11:07:48 GMT+0000 (Coordinated Universal Time)
cuid: clzgw0qjv000509m74ktcdq61
slug: building-a-robust-cicd-pipeline-with-github-actions-for-multi-cluster-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722855351929/abb3b921-9363-4866-b3b7-e3f40ed4dffe.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1722856058483/8a9eac72-6afd-49e0-a15e-cd3e4041550c.png
tags: docker, aws, github, java, sonarqube, kubernetes, maven, devops, eks, github-actions-1, trivy

---

In this project, we developed a robust Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Java-based web application. The application code is maintained in a GitHub repository with two main branches: `dev` and `main`. The `dev` branch is used for development purposes, while the `main` branch is used for production. Our goal was to automate the entire process of building, testing, and deploying the application to two distinct Kubernetes clusters: a development cluster (`dev cluster`) and a production cluster (`prod cluster`).

#### Technologies Used:

* **GitHub Actions:** For automating the CI/CD pipeline.
    
* **Maven:** For building the Java application.
    
* **Trivy:** For scanning the code and Docker images for vulnerabilities.
    
* **SonarQube:** For analyzing code quality, code smells, and code coverage.
    
* **Docker:** For containerizing the application.
    
* **Docker Hub:** For storing the Docker images.
    
* **Kubernetes (Kubeadm and EKS):** For deploying the application on development and production clusters.
    

**Infrastructure Required:**

* Github Runner: t2.medium instance with 12 GB storage.
    
* Setup master-worker(dev) kubeadm cluster for Dev branch.
    
* Setup EKS cluster(prod) for main branch.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722854092957/228cb525-9dee-4e94-b599-3ea5c31c1a06.png align="center")
    

### Detailed Explanation:

1. **SOURCE CODE MANAGEMENT:**
    

The project repository on GitHub contains two branches:

Github Repo:- [https://github.com/shubzz-t/Multi\_Cluster\_Java\_Project.git](https://github.com/shubzz-t/Multi_Cluster_Java_Project.git)

* `dev`: Contains code that is under development and is deployed to the development cluster which is kubeadm.
    
* `main`: Contains stable code that is ready for production and is deployed to the production cluster which is EKS cluster.
    

2. **RUNNER CONFIGURATION AND TOOLS INSTALLATION:**
    
    * **Configuring Runner:**
        
        1. Github &gt; Repository &gt; Settings &gt; Actions &gt; Runners &gt; New self-hosted runner &gt; Select OS (Linux)
            
        2. Run the commands we get onto the runner ec2 and it will be available.
            
    * **Installing tools:**
        
        1. Install Maven:
            
            ```bash
            #Install maven 
            sudo apt install maven
            ```
            
        2. Install Trivy:
            
            ```bash
            sudo apt-get install wget apt-transport-https gnupg lsb-release
            
            wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
            
            echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
            
            sudo apt-get update
            sudo apt-get install trivy
            ```
            
        3. Install docker and start sonarqube container:
            
            ```bash
            sudo apt update
            sudo apt install docker.io -y
            sudo chmod 666 /var/run/docker.sock
            
            #Creating/Running the sonarqube container
            docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
            ```
            
3. **CREATING SECRETS IN GITHUB REPOSITORY:**
    
    We need to create the secrets in github repository &gt; Settings &gt; Secrets and Variables &gt; Repository Secrets
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722851264403/af2a8ce0-7afe-453a-8aeb-59f3773e556b.png align="center")
    
    * Sonar credentials:
        
        1. SONAR\_TOKEN : Get this from sonarqube &gt; Administration &gt; Users &gt; Token name &gt; Generate.
            
        2. SONAR\_HOST\_URL : URL where sonarqube is running ex: http://10.10.10.10:9000
            
    * Docker credentials:
        
        1. DOCKERHUB\_USERNAME: Username for the dockerhub account.
            
        2. DOCKERHUB\_TOKEN: Password for the dockerhub account.
            
    * Kubernetes Cluster:
        
        1. KUBE\_CONFIG: From kubeadm master node run below command and paste the output as token in secrets.
            
            ```bash
            base64 ~/.kube/config
            ```
            
        2. KUBE\_CONFIG\_PROD: From EKS control plane run below command and paste the output as token in secrets for EKS cluster.
            
            ```bash
            base64 ~/.kube/config
            ```
            
4. **GITHUB WORKFLOW FOR DEV BRANCH:**
    
    This workflow will ensure that code changes are seamlessly built, tested, and deployed to the development environment, providing an efficient pipeline for continuous integration.
    
    ```yaml
    name: CICD
    
    on:
      push:
        branches: [ "Dev" ]
      pull_request:
        branches: [ "Dev" ]
    
    jobs:
      build:
    
        runs-on: self-hosted
    
        steps:
        - uses: actions/checkout@v4
        - name: Set up JDK 17
          uses: actions/setup-java@v3
          with:
            java-version: '17'
            distribution: 'temurin'
            cache: maven
    
        - name: Build with Maven
          run: mvn package --file pom.xml
    
        - name: Trivy FS scan
          run: trivy fs --format table -o fs.html .
    
        - name: SonarQube Scan
          uses: sonarsource/sonarqube-scan-action@master
          env:
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
    
        - name: Set up QEMU
          uses: docker/setup-qemu-action@v3
    
        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v3
    
        - name: Build Docker Image
          run: |
            docker build -t shubzz/devtaskmaster:latest .
    
        - name: Trivy Image scan
          run: trivy image --format table -o image.html adijaiswal/devtaskmaster:latest
    
        - name: Login to Docker Hub
          uses: docker/login-action@v3
          with:
            username: ${{ secrets.DOCKERHUB_USERNAME }}
            password: ${{ secrets.DOCKERHUB_TOKEN }}
    
        - name: Push Docker Image
          run: |
              docker push shubzz/devtaskmaster:latest
    
        - name: Kubectl Action
          uses: tale/kubectl-action@v1
          with:
            base64-kube-config: ${{ secrets.KUBE_CONFIG }}
        - run: |
              kubectl apply -f deployment.yml
    ```
    
    * **Trigger Configuration:**
        
        Workflow triggers are events that cause a workflow to run.
        
        ```yaml
        on:
          push:
            branches: [ "Dev" ]
          pull_request:
            branches: [ "Dev" ]
        ```
        
    * **Actions Configuration:**
        
        ```yaml
        jobs:
          build:
        
            runs-on: self-hosted
        ```
        
        This GitHub Actions configuration defines a build job that runs on a self-hosted runner (our runner ec2 in this case). This means the job will execute on a machine you manage, allowing for custom hardware and software configurations.
        
    * **Checkout code and JDK setup:**
        
        ```yaml
            - uses: actions/checkout@v4
            - name: Set up JDK 17
              uses: actions/setup-java@v3
              with:
                java-version: '17'
                distribution: 'temurin'
                cache: maven
        ```
        
        Uses the actions/checkout@v4 action to check out the repository code into the workflow.
        
        Uses the actions/setup-java@v3 action to set up JDK 17 with the Temurin distribution.
        
        Enables Maven dependency caching to speed up the build process.
        
    * **Maven and Trivy:**
        
        ```yaml
            - name: Build with Maven
              run: mvn package --file pom.xml
        
            - name: Trivy FS scan
              run: trivy fs --format table -o fs.html .
        ```
        
        Runs the Maven package command to build the project using the pom.xml file.
        
        Runs a Trivy file system scan for vulnerabilities, outputting the results in a table format to fs.html.
        
    * **Code Analysis SonarQube:**
        
        ```yaml
        - name: SonarQube Scan
              uses: sonarsource/sonarqube-scan-action@master
              env:
                SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
                SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        ```
        
        Uses the sonarsource/sonarqube-scan-action@master action to perform a SonarQube scan.
        
        SONAR\_TOKEN: Authentication token for SonarQube (stored securely in GitHub secrets).
        
        SONAR\_HOST\_URL: URL of the SonarQube server (stored securely in GitHub secrets).
        
    * **Setup QEMU and Docker Buildx:**
        
        ```yaml
          - name: Set up QEMU
              uses: docker/setup-qemu-action@v3
        
            - name: Set up Docker Buildx
              uses: docker/setup-buildx-action@v3
        
            - name: Build Docker Image
              run: |
                docker build -t shubzz/devtaskmaster:latest .
        ```
        
        Uses docker/setup-qemu-action@v3 to configure QEMU for emulating different CPU architectures in Docker builds.
        
        Uses docker/setup-buildx-action@v3 to enable Docker Buildx, allowing for advanced Docker builds including multi-platform support.
        
        Runs the docker build command to create a Docker image tagged shubzz/devtaskmaster:latest from the Dockerfile in the current directory.
        
    * **Kubectl Action and Deployment:**
        
        ```yaml
        - name: Kubectl Action
              uses: tale/kubectl-action@v1
              with:
                base64-kube-config: ${{ secrets.KUBE_CONFIG }}
            - run: |
                  kubectl apply -f deployment.yml
        ```
        
        Uses tale/kubectl-action@v1 to configure kubectl with a base64-encoded Kubernetes config file stored in GitHub secrets.
        
        Runs kubectl apply -f deployment.yml to apply the Kubernetes deployment configuration specified in deployment.yml.
        
5. **GITHUB WORKFLOW FOR MAIN BRANCH:**
    
    This workflow is designed to automate the CI/CD pipeline for the production environment, ensuring that code changes are thoroughly tested and deployed to the production cluster, maintaining a stable and reliable application.
    
    ```yaml
    name: CDPROD
    
    on:
      push:
        branches: [ "main" ]
      pull_request:
        branches: [ "main" ]
    
    jobs:
      build:
    
        runs-on: self-hosted
    
        steps:
        - name: Login to Docker Hub
          uses: docker/login-action@v3
          with:
            username: ${{ secrets.DOCKERHUB_USERNAME }}
            password: ${{ secrets.DOCKERHUB_TOKEN }}
    
        - name: Push Docker Image
          run: |
              docker pull shubzz/devtaskmaster:latest
              docker tag shubzz/devtaskmaster:latest shubzz/prodtaskmaster:latest
              docker push shubzz/prodtaskmaster:latest
        - name: Kubectl Action
          uses: tale/kubectl-action@v1
          with:
            base64-kube-config: ${{ secrets.KUBE_CONFIG_PROD }}
        - run: |
              kubectl apply -f deployment.yml
          
    ```
    
    * **Trigger Configuration:**The workflow runs on pushes or pull requests to the `main` branch.
        
    * **Build Job:** Runs on a self-hosted runner.
        
    * **Login to Docker Hub:** Authenticates to Docker Hub using credentials stored in GitHub secrets.
        
    * **Push Docker Image:** Pulls the latest image, tags it for production, and pushes it to Docker Hub.
        
    * **Kubectl Action:** Configures `kubectl` with a production Kubernetes config and applies the deployment using `kubectl`.
        

### **References:**

For an in-depth guide on setting up a CI/CD pipeline, I highly recommend this YouTube [Project by Devops Shack](https://www.youtube.com/watch?v=YxXlneeJkO4). A big thank you to Devops Shack for the excellent tutorial!

### **Conclusion:**

This CI/CD pipeline automates the process for both the `dev` and `main` branches, from building and testing to deploying to development and production clusters. Leveraging GitHub Actions, Maven, Trivy, SonarQube, Docker, and Kubernetes ensures a consistent, efficient, and secure deployment workflow.

Thank you for taking the time to read my blog on setting up a CI/CD pipeline with GitHub Actions. I hope you found it informative and helpful.

If you have any questions or need further clarification on any part of the project, feel free to reach out. I'm here to help!
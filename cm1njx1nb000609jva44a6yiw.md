---
title: "Building Secure Three-Tier Applications: DevSecOps on AWS with Kubernetes, GitOps, and ArgoCD"
datePublished: Sun Sep 29 2024 12:22:48 GMT+0000 (Coordinated Universal Time)
cuid: cm1njx1nb000609jva44a6yiw
slug: building-secure-three-tier-applications-devsecops-on-aws-with-kubernetes-gitops-and-argocd
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1727557444290/da93b8d7-d57e-4253-8893-10847c8b21d7.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1727612556801/67cba0dd-07ba-4e30-85c2-7ea0757fea95.png
tags: dns, docker, aws, kubernetes, devops, terraform, prometheus, grafana, eks, ci-cd, argocd, sonarcloud

---

In this blog, we will walk through the process of deploying a **three-tier quiz application** using **Terraform**, **GitHub Actions**, and **Amazon EKS (Elastic Kubernetes Service)**. The application is built with **React** for the frontend, **Node.js** for the backend, and **MongoDB** as the database, representing a modern stack for web application development.

We’ll explore how to automate infrastructure provisioning on AWS with Terraform, set up continuous integration (CI) and continuous delivery (CD) pipelines using GitHub Actions, and deploy the application to a scalable Kubernetes cluster. Along the way, we'll integrate essential security and monitoring tools like **SonarCloud**, **Snyk**, **Trivy**, **Prometheus**, and **Grafana** to ensure code quality, security, and performance visibility.

### 1\. Prerequisites

* AWS Account, Github Account
    
* AWS User: Create aws user with administrator access(for practice purpose).
    
* AWS S3 bucket for terraform backend.
    
* IAC git repository: [https://github.com/shubzz-t/IAC\_VPC\_with\_Jumphost.git](https://github.com/shubzz-t/IAC_VPC_with_Jumphost.git)
    
* Source Code Git repository: [https://github.com/shubzz-t/React\_3Tier\_Quiz\_App.git](https://github.com/shubzz-t/React_3Tier_Quiz_App.git)
    

### 2\. Infra by Terraform via Github Actions Setup

* Fork the repository for the terraform code.
    
* Go to repository &gt; Settings &gt; Secrets and variables &gt; Add AWS\_SECRET\_ACCESS\_KEY and AWS\_ACCESS\_KEY\_ID.
    
* Creating and adding the bucket name as secret BUCKET\_TF\_STATE.
    
* Pushing the code to the github will trigger the github workflow and terraform will provision the infrastructure on AWS.
    

### 3\. CI setup:

* **Fork the source code repository.**
    
    Source code contains code for frontend, backend, CI workflow and kubernetes manifests.
    

* **Configure Github Token:**
    
    1. Copy the github login token and add it to the github secrets.
        
* **Configure Sonar Cloud:**
    
    1. Login to sonar cloud and create new organization.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727516232505/2539d8b0-ee14-4942-8a8c-a4f439e53db9.png align="center")
        
    2. Enter the details for the organization.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727516329175/b0432a97-297c-47a3-b889-0705aa5264ad.png align="center")
        
    3. Click on analyze app and fill the details.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727516410366/88c87c76-a9a1-41ae-8195-11e4a33e1890.png align="center")
        
    4. Click Next. Then select previous version and create.
        
    5. Next in github create secrets for:
        
        SONAR\_ORGANIZATION= quiz-aap
        
        SONAR\_PROJECT\_KEY= quiz-aap
        
        SONAR\_URL= [https://sonarcloud.io/](https://sonarcloud.io/)
        
        SONAR\_TOKEN= &lt;your\_generated\_token&gt;
        
        Can create token by going to account &gt; settings &gt; security &gt; Generate token.
        
* **Configure Snyc:**
    
    Snyk integrates into all stages of development - IDEs, source code managers, CI/CD pipelines, and repositories, to detect high-risk code, open source packages, containers, and cloud configurations, and give developers the precise information they need to fix each issue - or even to automate the fix, where desired.
    
    1. Login to Snyc.
        
    2. Go to Account Settings &gt; Copy token.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727518947799/4f6338f2-d752-43f7-a25d-852cee5bc4fa.png align="center")
        
    3. Add the token in github secrets for:
        
        SNYK\_TOKEN= &lt;your\_generated\_token&gt;
        
* **Configure Docker:**
    
    Docker is a software platform that helps developers build, test, and deploy applications quickly and efficiently
    
    1. Login to docker.
        
    2. Create token by clicking account settings.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727518161321/7c763991-bd9c-49f1-bf60-ffd4c8b355bc.png align="center")
        
    3. Click on personal access tokens &gt; Create token &gt; Copy token.
        
    4. Create a secret in github for:
        
        DOCKER\_USERNAME= shubzz
        
        DOCKER\_PASSWORD= &lt;your\_generated\_token&gt;
        
* **The github workflow for the CI includes installing, scanning the code.**
    
    1. **Triggers:**
        
        The workflow is triggered on a **pull request** to the **main** branch.
        
    2. **Jobs:**
        
        * **Frontend Testing (frontend-test):**
            
            Uses Node.js (version 20.x) on an Ubuntu environment.
            
            Installs dependencies, runs **linting**, **prettier**, and **unit tests** for the frontend.
            
            Analyzes the code quality using **SonarQube** and **SonarCloud**.
            
        * #### **Backend Testing (backend-test)**:
            
            Similar to frontend, but applied to the backend.
            
            Runs linting, formatting, and testing.
            
            Conducts a **SonarQube** code analysis.
            
        * #### **Frontend Security (frontend-security)**:
            
            Depends on successful frontend testing.
            
            Uses **Snyk** to scan for security vulnerabilities in the frontend codebase.
            
        * #### **Backend Security (backend-security)**:
            
            Depends on backend testing.
            
            Uses **Snyk** to check for vulnerabilities in the backend code.
            
        * #### **Frontend Docker Image (frontend-image)**:
            
            Builds a Docker image for the frontend after passing security checks.
            
            Pushes the image to Docker Hub and uses **Trivy** and **Snyk** to scan for vulnerabilities in the Docker image.
            
        * #### **Backend Docker Image (backend-image)**:
            
            Builds and pushes a Docker image for the backend.
            
            Similar to frontend, scans the backend Docker image with **Trivy** and **Snyk**.
            
        * #### **Kubernetes Manifest Scan (k8s-manifest-scan)**:
            
            After backend and frontend security scans, runs **Snyk** to check Kubernetes manifest files for any security issues.
            
* After pushing the changes to the main branch the CI workflow will start running.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727523838555/73ccbe25-7b65-4b27-aa10-8ec89805caeb.png align="center")
    

### 4\. CD Setup:

* **Login to the Jump server provisioned using the terraform.**
    
    SSH into the server using:
    
    ```bash
    ssh -i mykey.pem ubuntu@<public_ip_jump_server>
    ```
    
* **Create EKS cluster:-**
    
    Create EKS cluster with 2 nodes to deploy the application to create EKS cluster use command:
    
    ```bash
    eksctl create cluster --name my-eks-cluster --region ap-south-1 --node-type t2.medium --nodes 3
    ```
    
* **Install Load Balancer Controller:**
    
    Create OIDC provider and Service account attach policy to service account and create Load Balancer Controller.
    
    1. Create OpenID Connect (OIDC):
        
        ```bash
        eksctl utils associate-iam-oidc-provider \
            --region ap-south-1 \
            --cluster my-eks-cluster \
            --approve
        ```
        
    2. Create Policy:
        
        ```bash
        #Download policy using
        curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.1/docs/install/iam_policy.json
        
        #Create policy
        aws iam create-policy \
            --policy-name AWSLoadBalancerControllerIAMPolicy \
            --policy-document file://iam-policy.json
        ```
        
    3. Create a IAM role and Service Account for the AWS Load Balancer controller:
        
        ```bash
        eksctl create iamserviceaccount \
        --cluster=my-eks-cluster \
        --namespace=kube-system \
        --name=aws-load-balancer-controller \
        --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
        --override-existing-serviceaccounts \
        --approve
        ```
        
    4. Install Helm:
        
        ```bash
        sudo snap install helm --classic
        ```
        
    5. Add eks chart repo to helm:
        
        ```bash
        helm repo add eks https://aws.github.io/eks-charts
        
        #Helm repo update
        helm repo update eks
        ```
        
    6. Install Load Balancer Controller:
        
        ```bash
        helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
        
        #Can verify using command
        kubectl get deployment -n kube-system aws-load-balancer-controller
        ```
        
* **Install Argo CD:**
    
    1. Apply the manifest:
        
        ```bash
        kubectl apply -n argocd -f 
        https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/install.yaml
        ```
        
    2. Change NodePort:
        
        ```bash
        kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
        ```
        
    3. Get ArgoCD password:
        
        ```bash
        kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
        ```
        
    4. Access the argoCD on load balancer dns name you will get the argo CD login page.
        
        Username= admin
        
        Password= &lt;output of above command&gt;
        
* **Add Monitoring Tools:**
    
    1. Add Repository:
        
        ```bash
        #Create Namespace for monitoring
        kubectl create namespace monitoring
        
        #Adding repo
        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        
        #Update Helm repo
        helm repo update
        ```
        
    2. Install Prometheus and Grafana:
        
        ```bash
        helm install monitoring prometheus-community/kube-prometheus-stack
        ```
        
    3. Edit prometheus and grafana service from Cluster IP to Load Balancer:
        
        ```bash
        kubectl edit svc <service_name> -n <namespace>
        ```
        
    4. Access the prometheus on &lt;LoadBalancerIP&gt;:9090 and Grafana on &lt;LoadBalancerIP&gt; using username=admin and password you can get using command:
        
        ```bash
        kubectl get secret --namespace monitoring <secret name> -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
        ```
        
    5. Configure Prometheus as Datasource in Grafana &gt; Datasource and provide prometheus access url.
        
    6. Next create Dashboard go to Dashboard &gt; Click new &gt; Import &gt; Enter 15757 as dashboard code and click load and Import.
        
* **Configure ArgoCD:**
    
    1. Click on create application.
        
    2. Give application name, project name as default, sync policy as automatic.
        
    3. Source as git repo url
        
    4. Path as kubernetes-manifest i.e the path where manifest are stored.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727608620439/409a139f-3158-4373-a5a3-4b8a5764ab21.png align="center")
        
    5. Cluster url default and namespace as quiz.
        
    6. Click create.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727608439415/01c62191-8729-4e39-8018-fc36e51da91d.png align="center")
        
* **Configure Route 53:**
    
    1. Create A record in route 53 pointing to the Load Balancer of the Frontend application. Create record.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727607658419/1ade7f30-9f56-4bbd-bae1-297468caddb9.png align="center")
        
    2. Now you will be able to access your website on the configured domain name.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727607954264/b5684305-4919-4cec-a55a-12e72a10f07d.png align="center")
        
* **Note:** The website is no longer available on the mentioned domain as all resources were deleted after the project's completion to avoid ongoing costs.
    

## Conclusion

By following this guide, you’ve deployed a **three-tier quiz application** using **React**, **Node.js**, and **MongoDB** on **Amazon EKS**, with automated infrastructure via **Terraform** and **CI/CD pipelines** using **GitHub Actions**. Integrating tools like **SonarCloud**, **Snyk**, **Trivy**, **Prometheus**, and **Grafana** ensures your app is secure, monitored, and scalable.

This setup streamlines deployment, boosts security, and enhances performance monitoring, providing a solid foundation for handling production workloads efficiently.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
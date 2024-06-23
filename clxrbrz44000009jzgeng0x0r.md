---
title: "Enhancing Fleet Management : A Deep Dive into Fleetman App Deployment"
datePublished: Sun Jun 23 2024 09:07:10 GMT+0000 (Coordinated Universal Time)
cuid: clxrbrz44000009jzgeng0x0r
slug: enhancing-fleet-management-a-deep-dive-into-fleetman-app-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719133541875/c19e2271-23e5-4583-b485-82851315b393.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719133619997/5acea257-a385-41bd-aebf-c1e2c6f4fca1.png
tags: microservices, docker, kubernetes, devops, jenkins

---

In the fast-paced landscape of modern fleet management, precision and real-time data are paramount. Meet Fleetman, an innovative application revolutionizing how vehicle fleets are monitored and managed. Powered by a sophisticated microservices architecture, Fleetman integrates cutting-edge technologies to provide seamless tracking, data management, and insightful analytics.

At its core, Fleetman comprises several specialized microservices, each playing a crucial role in ensuring operational excellence. The journey begins with the **Simulator Service**, capturing real-time vehicle positions across bustling city streets. This data is swiftly relayed to the **Apache MQ**, a robust message queue microservice that efficiently manages message flow to and from the **Position Tracker Service**. Here, vehicle positions are meticulously tracked and stored in the **MongoDB Database Service**, ensuring persistent and reliable data storage.

Facilitating seamless data access and integration is the **API Gateway Service**, which acts as the gateway to retrieve vehicle tracking information from the Position Tracker Service. Finally, presenting these insights in a user-friendly interface is the **Front-end Service**, crafted with Angular to visualise and interpret the tracked paths and locations of fleet vehicles.

Let's dive into the blog, where we unravel the technological advancements driving Fleetman's impact on fleet management.

**References**[: //www.udemy.com/course/kubernetes-microservices/](https://www.udemy.com/course/kubernetes-microservices/)

> **Note:** If you follow the course then no need to clone the below repository. I have forked repository and changed few code to work on EKS while the app is deployed on minikube in course.

**GitHub Repository:**[https://github.com/orgs/shubz-fleetman-org/repositories](https://github.com/orgs/shubz-fleetman-org/repositories)

**Prerequisites:**

* EKS Cluster with minimum 2 t2.medium nodes.
    
* Clone code from repository for jenkins : [https://github.com/shubz-fleetman-org/jenkins](https://github.com/shubz-fleetman-org/jenkins)
    

### Step 1: Creating the jenkins pod:

* You will get 2 files inside cloned jenkins repository. 1) Dockerfile 2) jenkins.yml
    
* Apply the jenkins.yml to create the pod for the jenkins image.
    
    ```bash
    kubectl apply -f jenkins.yml
    ```
    
* Next, check on which node is the jenkins pod running so that we can access jenkins using address as : node\_public\_ip:nodeport
    
* You can check node using the below command:
    
    ```bash
    kubectl get pods -o wide
    ```
    
* Now you can access the jenkins home page.
    

### Step 2: Creating pipeline

* Firstly set the ORGANIZATION\_NAME environment variable. Click Manage Jenkins &gt; System &gt; Environment variable &gt; Add &gt; Name as ORGANIZATION\_NAME and value as shubz-fleetman-org in my case.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719128141245/51cdfe44-4667-4782-b355-664ce59c24c5.png align="center")
    
* Next on dashboard click on new item to create new jenkins job.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719126976839/11d2182e-bb53-4860-83bc-c0eb701bb2dd.png align="center")
    
    Enter the project name (any-name) select organisation folder as we are going to run all repositories inside organisation. Click OK.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719127099492/a06c19fb-7ed7-4ecf-ab92-53b0d89acbd3.png align="center")
    
    Next you will get the above page select github organization under projects section.
    
* Next, under credentials section click on add dropdown will appear select Jenkins from that to enter credentials for github account.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719127367351/e72a4e2f-de22-481b-b799-074702066db3.png align="center")
    
    Next enter the username for you account in my case I have entered mine and the password as your github token. Enter ID it can be any and the description if you want to.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719127518371/de5a834f-3554-447a-91aa-633548fc27cc.png align="center")
    
    Next select our entered credentials from the dropdown.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719127596935/3161326a-c4ca-4800-9e8a-0f53bbc36dce.png align="center")
    
    Enter the organization name in the owner section.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719127669679/66c866df-18d6-4889-b607-6307b63adb87.png align="center")
    
    Click on Apply and Save.
    
* Jenkins will start scanning your organization for the repositories and the Jenkinsfile to build and deploy the code/images.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719127806376/c73eb617-0c0b-4353-822e-fb4327b215c2.png align="center")
    

### Step 3: Accessing the application

* Use the command `kubectl get pods -o wide` to find the node on which your web application (the Frontend Micro service) is running. This command provides essential information about the pods, including their status and the node they are deployed on.
    
* For example, if your node's IP address is node\_public\_ip and the NodePort assigned to your frontend service is nodeport, you would enter `node_public_ip:nodeport` into your web browser.
    

In this blog post, we've explored the deployment journey of Fleetman, a cutting-edge application revolutionizing fleet management through advanced microservices architecture.

If you have any questions, need further clarification, or just want to share your DevOps success stories, feel free to reach out. I'm here to help and eager to hear about your DevOps adventures. Drop me a message, and let's keep the conversation going. Happy coding!
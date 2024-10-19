---
title: "AWS CI/CD with AWS CodeCommit, CodeBuild, CodeDeploy, and CodePipeline with Rollback Capability"
seoTitle: "AWS Code Pipeline"
datePublished: Sat Oct 19 2024 09:21:27 GMT+0000 (Coordinated Universal Time)
cuid: cm2fy8uwo000d09jx3131cxpd
slug: aws-cicd-with-aws-codecommit-codebuild-codedeploy-and-codepipeline-with-rollback-capability
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729329448042/8fc11136-9077-4593-a4b2-d1e3c5c803ef.gif
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1729329597595/51494b2d-b4a4-44c8-a41b-35df3208de36.gif
tags: ec2, aws, devops, cloud9, s3, codedeploy, codepipeline, route53, codecommit, codebuild, aws-code-artifact

---

In this project, I built an end-to-end CI/CD pipeline utilizing AWS DevOps services such as Cloud9, CodeCommit, CodeBuild, Code Artifactory, CodeDeploy, CodePipeline and Route 53. The pipeline is designed to automate the entire software delivery process, from committing code to deploying the application. Artifacts are stored in S3 for version control, and automatic rollback mechanisms are in place to ensure stability in case of deployment failures. This project demonstrates the power of AWS native tools in creating scalable, reliable, and efficient CI/CD workflows, ensuring continuous integration and delivery with minimal manual intervention.

Prerequisites:

* AWS Account with S3 bucket.
    
* Any basic java project. Can have this one [https://github.com/shubzz-t/AWS-Code-Pipeline.git](https://github.com/shubzz-t/AWS-Code-Pipeline.git)
    

### 1\. Creating Project Cloud9

* **Create Cloud9 Environment**:
    
    1. Navigate to Cloud9 and click Create environment.
        
    2. Name your environment (e.g., `my-maven-project-env`), and select the instance type (e.g., `t2.micro`).
        
    3. Set other parameters as needed and click Create.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729276392867/4dac8e56-d798-4b4b-987c-c43c022d3e00.png align="center")
        
* **Install Maven and Java**:
    
    * Once the environment is ready, open the terminal in Cloud9 and run the following commands to install Maven and Java:
        
        ```plaintext
        sudo yum install maven -y
        sudo yum install java-1.8.0-openjdk -y
        ```
        
* **Generate Maven Project**:
    
    * Run the following Maven command to generate your project with a specific archetype:
        
        ```plaintext
        mvn archetype:generate \
            -DgroupId=com.myproject.app \
            -DartifactId=myproject-web-app \
            -DarchetypeArtifactId=maven-archetype-webapp \
            -DinteractiveMode=false
        ```
        
    * This will create a basic Maven web project under `myproject-web-project`. You can now navigate to the directory and start building your application.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729277257259/72862934-bace-4ad9-a5a7-015b8856a4e7.png align="center")
        

### 2\. Push code to Code Commit:

* **Create CodeCommit Repository**:
    
    * In the AWS Console, navigate to CodeCommit and click Create repository.
        
    * Provide a repository name (e.g., `my-maven-web-project`), and add a description if needed.
        
    * Click Create to generate the repository.
        
* **Configure Git**:
    
    * In your Cloud9 terminal (or local system), configure your Git user details:
        
        ```plaintext
        git config --global user.name "your-username"
        git config --global user.email "your-email@example.com"
        ```
        
* **Initialize Git in the Maven Project**:
    
    * Navigate to your Maven project directory and initialize Git:
        
        ```plaintext
        cd myproject-web-project
        git init
        ```
        
* **Add CodeCommit Repository as Remote**:
    
    * Copy the HTTPS or SSH URL of the CodeCommit repository.
        
    * Add it as a remote origin:
        
        ```plaintext
        git remote add origin https://git-codecommit.<region>.amazonaws.com/v1/repos/my-maven-web-project
        ```
        
* **Add and Push Code to CodeCommit**:
    
    * Add all the project files to the repository:
        
        ```plaintext
        git add .
        git commit -m "Initial commit"
        ```
        
    * Push the code to the `main` branch of CodeCommit:
        
        ```plaintext
        git push origin main
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729278290149/c7f5b432-283c-4642-9efc-66efe96de743.png align="center")
        

### 3\. Configure Code Artifactory Repository:

* **Create CodeArtifact Repository**:
    
    * In the AWS Console, navigate to CodeArtifact and click Create repository.
        
    * Provide a repository name (e.g. `my-maven-artifact-repo`), and select the appropriate domain if needed.
        
    * Click Create to generate the repository.
        
* **View Connection Instructions**:
    
    * After the repository is created, click View connection instructions at the top.
        
    * Choose Mac and Linux as the platform and Maven as the tool.
        
    * Follow the on-screen instructions.
        
* **Copy Connection Commands**:
    
    * Copy the provided commands from the View connection instructions section.
        
    * Paste the export command into your terminal to configure your Maven settings for CodeArtifact.
        
* **Create Maven** `settings.xml`:
    
    * Navigate to your project directory:
        
        ```plaintext
        cd ~/environment/myproject-web-project
        ```
        
    * Create a `settings.xml` file:
        
        ```plaintext
        echo $'<settings>\n </settings>' > settings.xml
        ```
        
    * Copy the steps 4,5,6 from the instructions and paste it in the settings.xml.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729282218896/a33652f7-4e81-4dde-83ff-f4002b192ca7.png align="center")
        
    * Check weather artifact repository is connected with the maven using the command.
        
        ```plaintext
        mvn -s settings.xml compile
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729282350290/f65012a7-eece-4f6e-ae52-1942b14eebff.png align="center")
        

### 4\. Code Build Configuration:

* **Project Name and Tags**:
    
    * Go to the CodeBuild console and click Create build project.
        
    * Enter the project name as myproject.
        
    * (Optional) Add tags for better project management and categorization.
        
* **Source**:
    
    * For the source provider, select CodeCommit.
        
    * Choose the repository that you created earlier.
        
    * Select the master branch.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729282455301/96cb7655-6c25-46ab-be66-63d4a198173a.png align="center")
        
* **Environment**:
    
    * Provisioning: Ondemand
        
    * Environment image: Choose Managed image.
        
    * Operating system: Select Amazon Linux 2. Runtime: standard.
        
    * Runtime: Select Corretto (Java) and choose the latest version available.
        
    * Service role: Select Create new service role if you don’t have one already.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729282574941/f8711091-c1f2-4e4d-960c-65d55bef47f4.png align="center")
        
* **Buildspec**:
    
    * Select Use a buildspec file. CodeBuild will look for the `buildspec.yml` file in your source code repository.
        
* **Artifacts**:
    
    * Artifact type: Choose Amazon S3.
        
    * S3 Bucket: Select your previously created bucket ie: myawscicdartifact .
        
    * Artifact name: Set the name to [`myproject.zip`](http://projectname.zip).
        
    * Change the packaging option to ZIP so the output will be bundled in a zip file.
        
* **Logs**:
    
    * Enable CloudWatch logs.
        
    * CloudWatch log group name: Set the name to `myproject-logs`.
        
    * Stream name prefix: Use `webapp` for easier identification.
        
* **Create Build Project**:
    
    * Review the configurations and click Create build project.
        
* **Create role for CodeBuild**:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729283959557/4da0acf4-ee8c-4b16-a878-20bfb2477388.png align="center")
    
* **Create Build:**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729283863757/590454af-b402-4f32-aa49-91c32650bdb5.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729283888450/1dbdc3fe-d7d7-4a18-93a8-f690f1203da3.png align="center")
    

This setup configures CodeBuild to:

* Source code from your CodeCommit repository.
    
* Build using Amazon Linux with Corretto.
    
* Output a zipped artifact to Amazon S3.
    
* Log build activities to CloudWatch Logs for monitoring.
    

### 5\. Configure Code Deploy

* Create EC2 Instance:
    
    * Create ec2 instance from AWS console with t2 micro Amazon Linux image.
        
    * Give tag to the image.
        
* **Create a CodeDeploy Application**:
    
    * Go to the CodeDeploy console and click on Create application.
        
    * Give your application a name (e.g., `addition-application`).
        
    * For the Compute Platform, select EC2/On-premises.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729284234934/08a65c9e-36f7-482e-9e6a-e21874f1737d.png align="center")
        
* **Create Deployment Group**:
    
    * Click Create deployment group.
        
    * Name your deployment group (e.g., `myproject-deployment-group`).
        
    * Select the service role with the required permissions for CodeDeploy.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729284888278/ff78575a-18ca-4d29-a3d2-dcde68290445.png align="center")
        
    * For Deployment Type, choose In-place deployment.
        
    * Under Environment Configuration, select Amazon EC2 instances.
        
    * Choose the instances with the CodeDeploy agent installed.
        
    * Install AWS CodeDeploy Agent: Now and schedule updates
        
* **Update Scheduling**:
    
    * Under Automatic rollback, you can leave the defaults or modify the schedule to update every 14 days if needed.
        
    * For the deployment strategy, select All at once (default) or adjust as per your requirement.
        
    * Uncheck Enable load balancing if you don't plan to use it.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729285120630/75a7425a-c491-4042-a339-1abdbd278a9c.png align="center")
        
* **Create a Deployment**:
    
    * Go to Deployments and click on Create deployment.
        
    * Select S3 as the source for the application code.
        
    * Enter the S3 URI where your zip file is stored (e.g., `s3://yourbucketname/`[`projectname.zip`](http://projectname.zip)).
        
    * For the Revision type, ensure it is ZIP.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729285198498/fea09955-aa45-49e0-a5f0-ad32de476927.png align="center")
        
* **Launch Deployment**:
    
    * Review the configuration and click Create deployment.
        
    * CodeDeploy will deploy the application from the S3 zip file to your EC2 instances.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729289570715/ad29339d-ebf5-404a-84b7-47a930665404.png align="center")
        

### 6\. Configure Code Pipeline:

* **Create Pipeline**:
    
    * Give your pipeline a meaningful name.
        
* **Custom Template**:
    
    * Choose the option to create from a custom template.
        
* **Execution Mode**:
    
    * Select Superseded for execution mode, which allows for more flexible updates to your pipeline.
        
* **Service Role**:
    
    * Choose to create a new service role or select an existing one that has the necessary permissions.
        
* **Source Provider**:
    
    * Select CodeCommit as your source provider.
        
    * Choose your repository and the branch you want to deploy.
        
* **CloudWatch Events**:
    
    * For the event trigger, select CloudWatch Events to initiate the pipeline based on specific events.
        
* **Build Stage**:
    
    * In the build section, choose Other build providers and select CodeBuild.
        
    * Specify the CodeBuild project name and opt for Single Build.
        
* **Deploy Stage**:
    
    * For the deployment stage, select CodeDeploy.
        
    * Choose your previously created deployment group and provide the deployment name.
        
* Create Pipeline:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729291932687/5640008c-33c8-47c9-8dee-38497c7ba563.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729291937748/8e84a799-2a32-4b39-93d2-a591d76da692.png align="center")
    

### 7\. Route 53 Config:

* **Create a Hosted Zone**:
    
    * Go to the AWS Route 53 console.
        
    * In the navigation pane, click on Hosted Zones.
        
    * Click on Create Hosted Zone.
        
        * Domain Name: Enter your domain name (e.g., [`example.com`](http://example.com)).
            
        * Type: Select Public Hosted Zone.
            
        * Click Create.
            
* **Create an A Record (Address Record)**:
    
    * Inside the hosted zone, click Create Record.
        
    * Choose Simple Routing and click Next.
        
    * For Record Name: Enter the subdomain (if any), or leave it blank to use the root domain (e.g., `www` or just the root domain `techopsjourney.site`).
        
    * Record Type: Choose A – IPv4 Address.
        
    * Value: Enter the public IP address of your EC2 instance (e.g., `192.0.2.44`).
        
    * TTL (Time to Live): You can leave the default (e.g., 300 seconds).
        
    * Click Create Record.
        
    * Update Your Domain Name Registrar (if necessary).
        
    * Update the domain's NS (Name Server) settings with the NS records from Route 53.
        
    * DNS changes might take some time to propagate globally (usually up to 24-48 hours, but often much faster).
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729323491083/cc954674-e69e-41eb-86f4-1b8dfa2a03b1.png align="center")
        
* Now you can access your website on your configured domain name. Here it is www.techopsjourney.site.
    

## Conclusion:

This project showcases the power and flexibility of AWS DevOps services in building a fully automated CI/CD pipeline. By integrating AWS tools such as Cloud9, CodeCommit, Code Artifactory, CodeBuild, CodeDeploy, CodePipeline and Route 53, we streamlined the software delivery process, ensuring efficient and continuous integration, testing, and deployment. Storing artifacts in S3 ensures version control and traceability, while the automatic rollback mechanism enhances deployment reliability. This end-to-end pipeline not only reduces manual effort but also fosters a more scalable, secure, and stable development workflow, highlighting the potential of AWS for modern DevOps practices.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
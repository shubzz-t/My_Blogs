---
title: "Integrating Jenkins with SonarQube for Powerful Code Analysis"
datePublished: Sat Apr 27 2024 18:29:38 GMT+0000 (Coordinated Universal Time)
cuid: clvifrrb900070amndbw38pe4
slug: integrating-jenkins-with-sonarqube-for-powerful-code-analysis
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713784349148/edf3456a-1aae-4756-83e1-d03248f0d79a.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1714242568782/620c15c6-516e-4638-b460-955321e22788.jpeg
tags: sonarqube, devops, jenkins

---

Welcome to our blog! Today, we're going to learn how to link Jenkins with SonarQube for top-notch code analysis. Don't worry if you're new to this – we'll guide you through each step using easy-to-understand language.

We're keeping things local by using Jenkins and Docker on your own machine. Plus, we have set up SonarQube latest effortlessly with Docker. By the end, you'll see how this integration can supercharge your code quality assessment.

Ready to make your coding life easier? Let's get started!

**#Prerequisites:**

* Jenkins server running.
    
* SonarQube running.
    

1. **Jenkins: Install Sonar Scanner Plugin**
    
    * Open Jenkins
        
    * Click Manage Jenkins
        
    * Click Plugins -&gt; Search for Sonar Scanner and install.
        
2. **SonarQube: Create Project and Credentials:**
    
    * Login to SonarQube with username: admin and password: admin.
        
    * Next click Create Local Project -&gt; give name -&gt; select branch as main -&gt; next use global settings -&gt; and click create project.
        
    * Click locally -&gt; give token name ( any name ) -&gt; generate the token.
        
    * Copy the token and paste it in notepad.
        
3. **Jenkins: Add SonarQube credentials in Jenkins Credentials**
    
    * Open Jenkins -&gt; Click Manage Jenkins -&gt; Credentials -&gt; System -&gt; Global Credentials.
        
    * Select secret text from drop down -&gt; paste the token copied to notepad in text box -&gt; Id give any name.
        
    * Create.
        
4. **Jenkins: Add SonarQube Server:**
    
    * Click Manage Jenkins -&gt; Click System
        
    * Go To sonarqube server section
        
    * Tick Environment Variables checkbox
        
    * Give any name you want.
        
    * And give the URL of sonarqube i.e. IPofmachine/9000 as sonarqube is running on the docker container.
        
    * Next you need to select your authentication token saved in credentials.
        
    * Click Save
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713774231643/22e7af72-3217-4709-b727-8502d1880f93.png align="center")
        
5. **Jenkins: Configuration in Pipeline**
    
    * Under build environment section tick **Prepare SonarQube Scanner environment** in checkbox.
        
    * Dialog box will appear inside you need to provide the server authentication token. Select the token from the drop down name of your token created in step 4 will appear there.
        
6. **Jenkins: Setting the Properties:**
    
    * Under the Jenkins Job Pre Steps select **Execute SonarQube Scanner** from the drop down.
        
    * Under JDK select inherit from job.
        
    * Next in analysis properties add the below properties:
        
    * Change the project.Key, projectName according to your created project in sonarqube.
        
        ```plaintext
        sonar.projectKey=java
        sonar.projectName=java
        sonar.projectVersion=1.0
        sonar.language=java
        sonar.tests=src/test/java
        sonar.sources=src/main/java
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713782245368/f0b15639-7e94-47ed-a1da-fc2cca099c16.png align="center")
        
    * Boom, you have integrated Jenkins with SonarQube and ready to run your job.
        

In conclusion, integrating Jenkins with SonarQube opens up a world of possibilities for enhancing your code quality analysis. Through the steps outlined in this blog, you've learned how to seamlessly connect these two powerful tools, all within the comfort of your local environment. I hope this guide has been informative and helpful in your quest for better code quality. If you have any questions or feedback, feel free to reach out. Happy coding!
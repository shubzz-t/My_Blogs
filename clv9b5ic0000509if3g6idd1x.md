---
title: "Exploring Two Ways of Jenkins-GitHub Integration"
datePublished: Sun Apr 21 2024 09:10:26 GMT+0000 (Coordinated Universal Time)
cuid: clv9b5ic0000509if3g6idd1x
slug: exploring-two-ways-of-jenkins-github-integration
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713689050538/79c4d24e-d27b-48f0-a95b-428965ab8f17.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1713690615584/21d1c374-2981-4e58-bfd3-02c3ca9cc928.png
tags: github, devops, jenkins, cicd-cjy1vtdk2005kjjs17n8couc3

---

In this blog, we'll explore how to seamlessly connect your Jenkins server with your GitHub account for Continuous Integration and Continuous Deployment (CI/CD) processes. We'll delve into two straightforward methods for integrating Jenkins with GitHub, ensuring that your Jenkins pipeline automatically kicks off whenever there's a code commit to your GitHub repository. Let's dive in!

**\# Prerequisites:**

* Server(local machine) with Jenkins installed.
    
* Github account with demo repository.
    

We'll cover two methods for integrating Jenkins with GitHub. The first method involves using GitHub hook trigger for Git SCM polling, while the second method is through Poll SCM. Both approaches enable Jenkins to detect changes in your GitHub repository and trigger the pipeline accordingly.

1. **Method 1: Github Hook trigger for Git SCM Polling:**
    
    When Jenkins receives a GitHub push hook, GitHub Plugin checks to see whether the hook came from a GitHub repository which matches the Git repository defined in SCM/Git section of the job. If they match and this option is enabled, GitHub Plugin triggers a one-time polling on GITScm. When GITScm polls GitHub, it finds that there is a change and initiates a build. The last sentence describes the behaviour of Git plugin, thus the polling and initiating the build is not a part of GitHub plugin.
    
    * **Adding payload url to github repository:**
        
        1. Go To Github repository for which you are building the CICD pipeline
            
        2. Go to the repository settings. Then click webhooks and then click Add Webhook.
            
        3. Add a payload url:
            
            e.g. https://jenkins\_server\_ip:8080/github-webhook/
            
        4. Next select content-type as application/json.
            
        5. Other all keep default and click create Webhook.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713640890221/ce808058-b8b0-4385-9576-95cc88da5dd9.png align="center")
            
    * Next you have to check the Github hook trigger for GITScm polling under Build Trigger section.
        
    * Your Github is integrated with the jenkins and will initiate the pipeline when the changed code is pushed to github.
        
2. **Method 2: Poll SCM:**
    
    A feature in Jenkins that periodically checks the version control system for changes. It uses a cron: Cron is a time-based job scheduler in Unix-like operating systems to schedule and automate the execution of jobs periodically at fixed times or specific intervals.
    
    * **Install github plugin on Jenkins:**
        
        Open Jenkins -&gt; Click Manage Jenkins -&gt; Click Plugins -&gt; Click Available Plugins -&gt; Search for Github plugin and click Install.
        
    * **Generating ssh keys on jenkins server:**
        
        1. On Jenkins server (where jenkins is installed) create ssh keys using the command ssh-keygen
            
            ```bash
            #To go into home directory
            cd ~
            
            #Command to create the ssh keys
            ssh-keygen
            
            #To go inside .ssh to see generated keys
            cd .ssh
            ```
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713676873210/5f3876cb-a402-4c99-a1d1-ae9b8611d2a3.png align="center")
            
            Above are the files present in .ssh directory
            
            id\_rsa = private key
            
            id\_rsa.pub = public key
            
            From this private key will be with jenkins and the public key we will give to the github.
            
    * **Adding public key to github:**
        
        1. Go to Github settings -&gt; click SSH & GPG keys
            
        2. Click new ssh key -&gt; any title you can give -&gt; next copy paste the public key into the key section.
            
        3. Click Add SSH key to create key.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713677324881/f9bf481e-5bcd-413d-bb35-df264b1eb7b7.png align="center")
            
    * **Adding private key to Jenkins:**
        
        1. Go to Jenkins webapp -&gt; Click Manage Jenkins -&gt; Click System -&gt; then Global Credentials.
            
        2. Click Add Credentials
            
            Kind: ssh username with private key
            
            Id: any meaningful name
            
            Username: the jenkins user (user with which you are using jenkins).
            
            Private key: copy the created key id\_rsa file content directly to the text box.
            
        3. Click create.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713678269517/35612c84-eff1-43d3-a7b4-770e9f082991.png align="center")
            
    * **Creating Job by selecting Poll SCM:**
        
        1. Select the Poll SCM from the pipeline to trigger/notice the code changes on github.
            
        2. You can run the Poll SCM to check for the code changes on github using the cron by setting the time intervals.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713681840473/4236a40f-ac8d-4161-9228-8f6a804a4d5a.png align="center")
            
3. **Difference between Poll SCM and Webhook trigger:**
    
    * Polling requests are made by the the receiver(here jenkins)of the data updates while webhook requests are made by the source(here github repo) of the data.
        
    * Polling is set up to run at fixed intervals and runs whether there is a new event or not whereas webhooks are also automatically triggered when an event occurs.
        

In summary, integrating Jenkins with GitHub streamlines CI/CD workflows. Whether you opt for GitHub hook trigger or poll SCM, these methods ensure seamless pipeline triggering on code commits. Mastering these integrations enhances collaboration and accelerates deployment.So, go ahead, experiment, and witness the power of Jenkins-GitHub integration in action! Happy coding!
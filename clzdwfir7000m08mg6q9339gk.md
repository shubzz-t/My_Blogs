---
title: "Configuring CloudWatch Agent to Send NGINX Logs from EC2 to CloudWatch"
datePublished: Sat Aug 03 2024 08:55:59 GMT+0000 (Coordinated Universal Time)
cuid: clzdwfir7000m08mg6q9339gk
slug: configuring-cloudwatch-agent-to-send-nginx-logs-from-ec2-to-cloudwatch
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722663154636/7f22da0f-3726-4756-9c06-8de11bff77a0.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1722675349264/8ae59dd7-ebf3-4d0b-8913-fe985ae52787.png
tags: aws, monitoring, devops, cloudwatch, devops-articles

---

In this blog, we will guide you through the process of configuring the CloudWatch Agent to send NGINX logs from your EC2 instances to Amazon CloudWatch. This setup will enable you to monitor and analyze your NGINX logs in real-time, providing valuable insights into your web server's performance and activity.

We'll cover the steps to install and configure the CloudWatch Agent on your EC2 instances, and show you how to customize the log collection to include your NGINX log files. Whether you're new to AWS or looking to enhance your log monitoring capabilities, this guide will help you set up an effective logging solution with ease.

Let's get started on optimizing your AWS infrastructure with CloudWatch!

1. **Launch Ubuntu EC2 Instance:-**
    
    * **Log in to AWS Management Console** and navigate to the EC2 Dashboard.
        
    * **Launch an Instance:**
        
        * Click "Launch Instance." Select "Ubuntu Server AMI"
            
        * Choose instance type as t2.micro.
            
        * Enable "Auto-assign Public IP."
            
        * Enable port 22 for ssh, port 80 for accessing the web page in security group.
            
        * In the "User data" section, add the following script to update the package list and install NGINX.
            
            ```bash
            #!/bin/bash
            sudo apt-get update
            sudo apt-get install nginx -y
            echo "ANY MESSAGE YOU WANT ON BROWSER" >> /var/www/html/index.html
            ```
            
        * Create instance.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722665799666/0a785a35-06fd-425b-85ee-521cc26568a2.png align="center")
            
2. **Create/Assign IAM Role for EC2 with CloudWatch permissions:-**
    
    * Log in to the AWS Management Console and go to the IAM Dashboard.
        
    * Click "Roles" &gt; "Create role." &gt; Select "AWS service" &gt; "EC2." &gt; Click "Next: Permissions."
        
    * Search for *CloudWatchAgentServerPolicy* and select it.
        
    * Name the Role, e.g., *EC2CloudWatchAgentRole*.
        
    * Click "Create role."
        
    * **Assign the Role to Your EC2 Instance:**
        
        * Go to the EC2 Dashboard.
            
        * Select your instance, click "Actions" &gt; "Security" &gt; "Modify IAM role."
            
        * Choose *EC2CloudWatchAgentRole* and click "Update IAM role."
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722665917393/1ffc690a-bd72-4aa9-bf59-60d433863739.png align="center")
            
3. **Download/Install CloudWatch Agent package:-**
    
    * Use the following steps to download the CloudWatch agent package, SSH to Instance & Download CW Agent package.
        
    * **Download the cloudwatch agent package using:**
        
        ```bash
        sudo wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722666177302/6cd99b2a-2fa6-4c10-a5ae-cd4176066cee.png align="center")
        
    * **Install the downloaded package:**
        
        ```bash
        sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
        ```
        
    * **Update Packages & Install collectd: (This will take a few minutes if you haven’t updated your available updates prior).**
        
        ```bash
        sudo apt-get update && apt-get install collectd
        ```
        
4. **Create the CloudWatch Agent Configuration File:-**
    
    * Before running the CloudWatch agent on any servers, you must create a CloudWatch agent configuration file.
        
    * The agent configuration file is a JSON file that specifies the metrics and logs that the agent is to collect, including custom metrics.
        
    * The agent configuration file wizard, *amazon-cloudwatch-agent-config-wizard,* multiple configuration related questions.
        
    * We can check json configuration file under /opt/aws/amazon-cloudwatch-agent/bin/config.json
        
    * Creating configuration file:
        
        ```bash
        /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
        ```
        
    * We are using Linux OS and using EC2 so we will select 1 option:
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722668367937/a22eccb8-21c7-430d-8293-70280a60cdd7.png align="center")
        
    * Next user we will select root, StatsD daemon = no,
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722669321748/b59c3715-38cb-48b2-ad6c-33660e40080d.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722669418203/2735573b-7761-4113-96b2-c4af4b5a5f67.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722669484030/79e70e2f-6b88-456a-94fa-a45fd2f1be3b.png align="center")
        
    * Here next give the path for your log file to monitor in our case it is /var/log/nginx/access.log
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722669574569/2bb8c67b-a773-4bc0-9ddd-e3170af57a0a.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722669631376/89e43ba9-556e-45f0-b1a7-6364ddc36af0.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722669667327/144c6e95-686e-4c78-a587-38da4a272d85.png align="center")
        
5. **Checking status of CloudWatch Agent:**
    
    ```bash
    /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
    ```
    
    It will appear as stopped, as shown below:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722670014923/3a9eff94-858a-4459-87d1-5162a10e8448.png align="center")
    
    We will start the status of the CloudWatch Agent:
    
    ```bash
    /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
    ```
    
    Now, we will again check for the status after running above command.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722670132288/ae494d4d-2c06-4c57-a7cf-25229fca9e2e.png align="center")
    
    Boom!! our cloudwatch agent is ready to send our Nginx logs to cloudwatch agent on AWS.
    
6. **Verifying logs on CloudWatch:**
    
    * We will see log group created with access.log name on CloudWatch.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722670710927/09519f5f-fb51-4147-8cc0-52305b677e06.png align="center")
        
    * Inside log group you will get a log stream containing all our logs.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722670775495/e4534f4f-f2c4-4282-add0-c493e96a000c.png align="center")
        
    * Here you will get all your logs on the CloudWatch.
        

**Conclusion:** In this blog, we have walked through the steps to configure the CloudWatch Agent to send NGINX logs from your EC2 instances to Amazon CloudWatch. By launching an EC2 instance with NGINX installed and setting up an IAM role with the necessary permissions, you've enabled a robust logging solution that enhances your ability to monitor and analyze your server's performance in real-time.

We hope this guide has been helpful in setting up your EC2 and CloudWatch integration. If you have any questions or need further assistance, feel free to reach out me anytime. Happy monitoring!
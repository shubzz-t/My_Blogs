---
title: "AWS Three-Tier Architecture Explained: VPC, EC2, ALB, RDS, and More"
datePublished: Sat Oct 12 2024 12:53:11 GMT+0000 (Coordinated Universal Time)
cuid: cm265q6du000008jz7jlohn7w
slug: aws-three-tier-architecture-explained-vpc-ec2-alb-rds-and-more
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1728737206052/51931042-2029-42de-a5d8-65d38873006f.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1728737575533/cac9f995-2160-4393-9bab-f7be384c3a19.png
tags: aws, cloudfront, devops, rds, vpc, load-balancing, route53, auto-scaling-aws

---

In today's cloud-driven world, building scalable and resilient applications is crucial for ensuring high availability and seamless user experiences. In this blog, I'll walk you through the deployment of a three-tier web application architecture on Amazon Web Services (AWS). This project integrates essential AWS services such as VPC for secure networking, EC2 instances for the application servers, Route 53 for domain management, CloudFront for content delivery, an Application Load Balancer (ALB) for routing traffic, and RDS with a standby instance for a robust MySQL database.

The three-tier architecture consists of:

* **Presentation Tier**: React application hosted on NGINX, distributed via CloudFront for fast, global delivery.
    
* **Application Tier**: Node.js backend, hosted on EC2 instances, managed behind an ALB for fault tolerance and load balancing.
    
* **Database Tier**: Amazon RDS (MySQL) with a standby instance for automated failover and high availability.
    

Throughout this guide, we’ll explore how to set up and manage these components to create a scalable, highly available, and secure application. Stay tuned for a deep dive into the configuration, deployment, and monitoring setup!

Prerequisites:

* Clone project using url: [https://github.com/shubzz-t/aws-three-tier-app-deployment.git](https://github.com/shubzz-t/aws-three-tier-app-deployment.git)
    

### 1\. Creating VPC and its Components:

* The foundation of our AWS three-tier architecture begins with setting up a **Virtual Private Cloud (VPC)**. We'll configure the VPC with two Availability Zones (AZs) to ensure high availability and fault tolerance. Here's how it's done:
    
    1. **VPC**: Create a custom VPC that will house all our resources, isolating them in a secure virtual network. I used ap-south-1 Mumbai Region.
        
    2. **Subnets**: We'll create **6 subnets**:
        
        * **4 private subnets** (2 per Availability Zone): These will host the application servers and the database for security purposes.
            
        * **2 public subnets** (1 per Availability Zone): These will host public-facing resources like the ALB (Application Load Balancer) and NAT Gateway.
            
            After creating the public subnets, go to **Subnet settings** and **enable auto-assign public IP** by clicking on the public subnet and selecting the option. This ensures that instances launched in the public subnets are automatically assigned a public IP address, allowing them to communicate with the internet.
            
    3. **NAT Gateway**: Set up a **NAT Gateway** in one of the public subnets. This allows instances in the private subnets to connect to the internet (for updates and patches) without exposing them to inbound internet traffic.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728724000927/ab13cde5-1dd4-43b1-a53a-9ea49b4c6336.png align="center")
        

### 2\. Creating Security Groups:

* Bastion Host Security Group:
    
    1. Name: bastion-host-sg
        
    2. Inbound Rule: Allow **SSH (Port 22)** access from your specific IP address for secure connection.
        
* Presentation Tier ALB Security Group:
    
    1. Name: presentation-alb-sg
        
    2. Inbound Rule: Allow **HTTP (Port 80)** traffic from anywhere (`0.0.0.0/0`).
        
* Presentation Tier EC2 Security Group:
    
    1. Name: presentation-ec2-sg.
        
    2. Inbound Rules:
        
        * **SSH (Port 22)**: Allow traffic from the **Bastion Host's Security Group** (i.e. bastion-host-sg).
            
        * **HTTP (Port 80)**: Allow traffic from the **Load Balancer's Security Group** (i.e. presentation-alb-sg).
            
* Application Tier ALB Security Group:
    
    1. Name: application-alb-sg.
        
    2. Inbound Rules:
        
        * **HTTP (Port 80)**: Allow traffic **only** from the **Presentation Tier EC2 Instances' Security Group** (i.e. presentation-ec2-sg).
            
* Application Tier EC2 Security Group:
    
    1. Name: application-ec2-sg.
        
    2. Inbound Rules:
        
        * **SSH (Port 22)**: Allow traffic from the **Bastion Host Security Group** (i.e., `bastion-host-sg`).
            
        * **Custom TCP (Port 3200)**: Allow traffic from the **Application Tier Load Balancer Security Group** (i.e. `application-alb-sg`).
            
* Database Tier EC2 Security Group:
    
    1. Name: database-ec2-sg
        
    2. Inbound Rules:
        
        * MySQL/Aurora (Port 3306): Allow traffic from Bastion Host Security Group(i.e. bastion-host-sg)
            
        * MySQL/Aurora (Port 3306): Allow traffic from Application Tier EC2 Instance SG (i.e. application-ec2-sg).
            
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728725321961/19696e51-96e3-4168-87fa-e5aed159aa11.png align="center")
        

### 3\. Creating Bastion Host

* Bastion Host Creation:
    
    1. **Launch EC2 Instance:**
        
        * Go to the **EC2 Dashboard** and click **Launch Instance**.
            
    2. **Select AMI and Instance Type:**
        
        * Choose **Ubuntu Server** and select instance type (e.g., `t2.micro`).
            
    3. **Configure Instance:**
        
        * Select your **VPC** and a **Public Subnet**. Enable **Auto-assign Public IP**.
            
    4. **Security Group:**
        
        * Select **bastion-host-sg** (ensure it allows SSH from your IP). Connect Bastion Host
            
            ```plaintext
            ssh -i /path/to/your-key.pem ec2-user@your-public-ip
            ```
            

### 4\. Setting RDS MySQL:

We will create an RDS MySQL instance configured as the primary database with a standby instance for high availability. The standby instance will replicate data from the primary, ensuring redundancy and automatic failover in case of an outage.

* Creating Subnet Group:
    
    1. **Select VPC**: Choose our created VPC where your database instances will be hosted.
        
    2. **Choose Availability Zones**: Select the availability zones where you want your database instances to be deployed.
        
    3. **Select Private Subnets**: Choose the private subnets within the selected VPC and availability zones for the subnet group. Ensure that these subnets provide adequate resources and redundancy for your RDS instances.
        
* Creating RDS Instance:
    
    1. **Create Database Instance**: Choose MySQL as the database engine.
        
    2. **Select Template**: Choose the "Dev/Test" template for a non-production environment.
        
    3. **Multi-AZ Deployment**: Ensure the Multi-AZ instance option is selected for high availability.
        
    4. **DB Instance Identifier**: Enter a name for the DB instance, such as `dev-db-instance`.
        
    5. **Master Username**: Set the master username as `admin`.
        
    6. **Credentials Management**: Choose "Self-managed" and enter a secure password for the master user.
        
    7. **Leave Default Settings**: Keep the default settings for other configuration options.
        
    8. **Select VPC**: Choose the VPC you created earlier.
        
    9. **Select Subnet Group**: Select the previously created subnet group for the RDS instances.
        
    10. **Select Security Group**: Choose the `database-tier-ec2-sg` security group to control access.
        
    11. **Create**: Click on the "Create Database" button to provision the MySQL database instance.
        
* Access RDS EC2 from Bastion Host:
    
    1. SSH into the Bastion host using its public IP:
        
        ```plaintext
        ssh -i /path/to/key.pem ec2-user@<Bastion-Host-IP>
        ```
        
    2. Install MySQL client:
        
        ```plaintext
        sudo yum install mysql
        ```
        
    3. Connect to RDS:
        
        ```plaintext
        mysql -h <RDS-Endpoint> -P 3306 -u admin -p
        ```
        
    4. Enter the master password when prompted.
        
    5. Run the following SQL queries to create the database, user, and grant privileges
        
        ```plaintext
        CREATE DATABASE react_node_app;
        CREATE USER 'appuser'@'%' IDENTIFIED BY 'learnIT02#';
        GRANT ALL PRIVILEGES ON react_node_app.* TO 'appuser'@'%';
        FLUSH PRIVILEGES;
        ```
        
    6. Log out from the current session.
        
    7. Log in again using the newly created user.
        
        ```plaintext
        mysql -h <RDS-Endpoint> -P 3306 -u appuser -p
        ```
        
    8. Enter the password `learnIT02#` when prompted.
        
    9. Once logged in as the appuser run the below sql scripts:
        
        ```plaintext
        -- Create Tables
        CREATE TABLE `author` ( 
          `id` int NOT NULL AUTO_INCREMENT, 
          `name` varchar(255) NOT NULL, 
          `birthday` date NOT NULL, 
          `bio` text NOT NULL, 
          `createdAt` date NOT NULL, 
          `updatedAt` date NOT NULL, 
          PRIMARY KEY (`id`) 
        ) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci; 
        
        CREATE TABLE `book` ( 
          `id` int NOT NULL AUTO_INCREMENT, 
          `title` varchar(255) NOT NULL, 
          `releaseDate` date NOT NULL, 
          `description` text NOT NULL, 
          `pages` int NOT NULL, 
          `createdAt` date NOT NULL, 
          `updatedAt` date NOT NULL, 
          `authorId` int DEFAULT NULL, 
          PRIMARY KEY (`id`), 
          KEY `FK_66a4f0f47943a0d99c16ecf90b2` (`authorId`), 
          CONSTRAINT `FK_66a4f0f47943a0d99c16ecf90b2` FOREIGN KEY (`authorId`) REFERENCES `author` (`id`) 
        ) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci; 
        
        
        -- Restore Data
        INSERT INTO `author` VALUES (1,'J.K. Rowling (Joanne Kathleen Rowling)','1965-07-31','J.K. Rowling is a British author best known for writing the Harry Potter fantasy series. The series has won multiple awards and sold over 500 million copies, becoming the best-selling book series in history. Rowling has also written other novels, including The Casual Vacancy and the Cormoran Strike crime series under the pen name Robert Galbraith.','2024-05-29','2024-05-29'),(3,'Jane Austen','1775-12-16','Jane Austen was an English novelist known for her wit, social commentary, and romantic stories. Her six major novels, which explore themes of love, marriage, and money, have earned her a place as one of the greatest writers in the English language.','2024-05-29','2024-05-29'),(4,'Harper Lee','1960-07-11','Harper Lee was an American novelist best known for her Pulitzer Prize-winning novel To Kill a Mockingbird. The novel explores themes of racial injustice and the importance of compassion. Lee published a sequel, Go Set a Watchman, in 2015.','2024-05-29','2024-05-29'),(5,'J.R.R. Tolkien','1954-07-29','J.R.R. Tolkien was a British philologist and writer best known for his fantasy novels The Hobbit and The Lord of the Rings. Tolkien\'s works have had a profound influence on the fantasy genre and popular culture.','2024-05-29','2024-05-29'),(6,'Mary Shelley','1818-03-03','Mary Shelley was a British novelist, playwright, and short story writer, the daughter of Mary Wollstonecraft Godwin and the wife of poet Percy Bysshe Shelley. Frankenstein, or, The Modern Prometheus (1818) is her most famous work.','2024-05-29','2024-05-29'),(7,'Douglas Adams','1979-10-12','Douglas Adams was an English science fiction writer, satirist, humorist, dramatist, screenwriter, and occasional actor. He is best known for the Hitchhiker\'s Guide to the Galaxy comedy series, which inspired a radio comedy, several books, stage shows, comic books, a 1981 TV series, a 1984 video game, a 2005 feature film, and a 2008 sequel film.','2024-05-29','2024-05-29'); 
        
        INSERT INTO `book` VALUES (1,'Harry Potter and the Sorcerer\'s Stone','1997-07-26','On his birthday, Harry Potter discovers that he is the son of two well-known wizards, from whom he has inherited magical powers. He must attend a famous school of magic and sorcery, where he establishes a friendship with two young men who will become his companions on his adventure. During his first year at Hogwarts, he discovers that a malevolent and powerful wizard named Voldemort is in search of a philosopher\'s stone that prolongs the life of its owner.',223,'2024-05-29','2024-05-29',1),(3,'Harry Potter and the chamber of secrets','1998-07-02','Harry Potter and the sophomores investigate a malevolent threat to their Hogwarts classmates, a menacing beast that hides within the castle.',251,'2024-05-29','2024-05-29',1),(4,'Pride and Prejudice','1813-01-28','An English novel of manners by Jane Austen, first published in 1813. The story centres on the relationships among the Bennet sisters, in particular Elizabeth Bennet the middle daughter, and the wealthy Mr. Darcy. Austen satirizes the social classes of the English gentry through a witty and ironic narrative voice.',224,'2024-05-29','2024-05-29',3),(5,'Harry Potter and the Prisoner of Azkaban','1999-07-08','Harry\'s third year of studies at Hogwarts is threatened by Sirius Black\'s escape from Azkaban prison. Apparently, it is a dangerous wizard who was an accomplice of Lord Voldemort and who will try to take revenge on Harry Potter.',317,'2024-05-29','2024-05-29',1),(6,'Harry Potter and the Goblet of Fire','2000-07-08','Hogwarts prepares for the Triwizard Tournament, in which three schools of wizardry will compete. To everyone\'s surprise, Harry Potter is chosen to participate in the competition, in which he must fight dragons, enter the water and face his greatest fears.',636,'2024-05-29','2024-05-29',1),(7,'The Hitchhiker\'s Guide to the Galaxy','1979-10-12','A comic science fiction comedy series created by Douglas Adams. The story follows the comedic misadventures of Arthur Dent, a hapless Englishman, following the destruction of the Earth by the Vogons, a race of unpleasant bureaucratic aliens. Arthur escapes with his friend Ford Prefect, who reveals himself to be an undercover researcher for the titular Hitchhiker\'s Guide to the Galaxy, a galactic encyclopedia containing information about anything and everything.',184,'2024-05-29','2024-05-29',7),(8,'Frankenstein; or, The Modern Prometheus','1818-03-03','A Gothic novel by Mary Shelley that tells the story of Victor Frankenstein, a young scientist who creates a grotesque creature in an unorthodox scientific experiment. Frankenstein is horrified by his creation and abandons it, but the creature seeks revenge. The novel explores themes of scientific responsibility, creation versus destruction, and the nature of good and evil.',211,'2024-05-29','2024-05-29',6),(9,'The Lord of the Rings: The Fellowship of the Ring','1954-07-29','The first book in J.R.R. Tolkien\'s epic fantasy trilogy, The Lord of the Rings. The Fellowship of the Ring follows hobbit Frodo Baggins as he inherits the One Ring, an evil artifact of power created by the Dark Lord Sauron. Frodo embarks on a quest to destroy the Ring in the fires of Mount Doom, accompanied by a fellowship of eight companions.',482,'2024-05-29','2024-05-29',5);
        ```
        

### 4\. Presentation Tier:

* Create a **launch template** for the presentation layer EC2 instances:
    
    1. **Navigate to Launch Templates**: Go to the EC2 Dashboard, and in the left-hand menu, click on "Launch Templates."
        
    2. **Create Launch Template**:
        
        * **Template Name**: Enter a name (e.g., presentation-tier-template).
            
        * **AMI**: Choose **Amazon Linux 2 AMI (HVM)**.
            
        * **Instance Type**: Select the instance type (e.g., `t2.micro`).
            
        * **Key Pair**: Choose an existing key pair or create a new one.
            
        * **Security Groups**: Select the **Presentation EC2 security group**, allowing HTTP (port 80) for alb and SSH (port 22 from bastion).
            
    3. **Do Not Specify Subnet**: Leave the subnet field blank, as this will be handled by the Auto Scaling group later for proper distribution across Availability Zones.
        
    4. User Data:
        
        ```plaintext
        #!/bin/bash 
        # Update the package list and install NGINX 
        sudo yum update -y 
        sudo yum install nginx -y 
        
        # Start and enable NGINX 
        sudo systemctl start nginx 
        sudo systemctl enable nginx 
        
        # Fetch metadata token 
        TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") 
        
        # Fetch instance details using IMDSv2 
        INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" "http://169.254.169.254/latest/meta-data/instance-id") 
        AVAILABILITY_ZONE=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" "http://169.254.169.254/latest/meta-data/placement/availability-zone") 
        PUBLIC_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" "http://169.254.169.254/latest/meta-data/public-ipv4") 
        
        # Create a simple HTML page displaying instance details 
        sudo bash -c "cat > /usr/share/nginx/html/index.html <<EOF 
        <h1>Instance Details</h1> 
        <p><b>Instance ID:</b> $INSTANCE_ID</p> 
        <p><b>Availability Zone:</b> $AVAILABILITY_ZONE</p> 
        <p><b>Public IP:</b> $PUBLIC_IP</p> 
        EOF" 
        
        # Restart NGINX to ensure changes are applied 
        sudo systemctl restart nginx
        ```
        
* Create Target Group:
    
    1. **Navigate to Target Groups**: Go to the **EC2 Dashboard** and click on **Target Groups** in the left-hand menu.
        
    2. **Create Target Group**:
        
        * **Choose Target Type**: Select **Instances**.
            
        * **Target Group Name**: Enter `presentation-tg`.
            
        * **Protocol**: Select **HTTP**.
            
        * **Port**: Set to **80**.
            
        * **VPC**: Select your **created VPC**.
            
    3. **Health Check Settings**:
        
        * **Path**: Enter `/` for the health check endpoint.
            
        * Leave other health check settings as default.
            
    4. **Create Target Group**: Click **Create Target Group** to complete the setup.
        
* Creating Application Load Balancer:
    
    1. **Navigate to Load Balancers**: Go to the **EC2 Dashboard** and select **Load Balancers**.
        
    2. **Create Load Balancer**:
        
        * **Name**: Enter `presentation-alb`.
            
        * **Scheme**: Select **Internet-facing**.
            
        * **Availability Zones**: Choose the **Availability Zones** and their corresponding **public subnets**.
            
    3. **Assign Security Group**:
        
        * Select the **security group** created i.e presentation-alb-sg.
            
    4. **Configure Routing**:
        
        * **Target Group**: Associate the ALB with the **target group** `presentation-tg`.
            
    5. **Create Load Balancer**: Click **Create Load Balancer** to complete the process.
        
* Create Auto Scaling Group:
    
    1. **Name the Auto Scaling Group**: Enter the name as `presentation-asg`.
        
    2. **Select Launch Template**: Choose the **launch template** we created earlier.
        
    3. **VPC and Subnets**: Select our VPC and the **public subnets** where the instances will be launched.
        
    4. **Attach Load Balancer**:
        
        * Choose **Attach to an existing load balancer**.
            
        * Select our **target group** `presentation-tg`.
            
    5. **Enable Health Checks**: Turn on **load balancer health checks**.
        
    6. **Group Metrics**: Enable **group metrics collection** in **CloudWatch**.
        
    7. **Desired Capacity**:
        
        * Set **Desired Capacity** to 3, **Minimum** 2, **Maximum** `4`.
            
    8. **Scaling Policy**:
        
        * Select **Target tracking policy**.
            
        * Set **Average CPU Utilization** to `50%`.
            

### 4\. Application Tier:

* Create a **launch template** for the presentation layer EC2 instances:
    
    1. **Navigate to Launch Templates**: Go to the EC2 Dashboard, and in the left-hand menu, click on "Launch Templates."
        
    2. **Create Launch Template**:
        
        * **Template Name**: Enter a name (e.g., `application-tier-template`).
            
        * **AMI**: Choose **Amazon Linux 2 AMI (HVM)**.
            
        * **Instance Type**: Select the instance type (e.g., `t2.micro`).
            
        * **Key Pair**: Choose an existing key pair or create a new one.
            
        * **Security Groups**: Select the **Application EC2 security group**, allowing HTTP (port 80) and SSH (port 22 from bastion).
            
    3. **Do Not Specify Subnet**: Leave the subnet field blank, as this will be handled by the Auto Scaling group later for proper distribution across Availability Zones.
        
    4. User Data:
        
        ```plaintext
        #!/bin/bash 
        # Update package list and install required packages 
        sudo yum update -y 
        sudo yum install -y git 
        
        # Install Node.js (use NodeSource for the latest version) 
        curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash - 
        sudo yum install -y nodejs 
        
        # Install PM2 globally 
        sudo npm install -g pm2 
        
        # Define variables 
        REPO_URL="https://github.com/learnItRightWay01/react-node-mysql-app.git" 
        BRANCH_NAME="feature/add-logging" 
        REPO_DIR="/home/ec2-user/react-node-mysql-app/backend" 
        ENV_FILE="$REPO_DIR/.env" 
        
        # Clone the repository 
        cd /home/ec2-user 
        sudo -u ec2-user git clone $REPO_URL 
        cd react-node-mysql-app  
        
        # Checkout to the specific branch 
        sudo -u ec2-user git checkout $BRANCH_NAME 
        cd backend 
        
        # Define the log directory and ensure it exists 
        LOG_DIR="/home/ec2-user/react-node-mysql-app/backend/logs" 
        mkdir -p $LOG_DIR 
        sudo chown -R ec2-user:ec2-user $LOG_DIR
        
        # Append environment variables to the .env file
        echo "LOG_DIR=$LOG_DIR" >> "$ENV_FILE"
        echo "DB_HOST=\"<rds-instance.end.point.region.rds.amazonaws.com>\"" >> "$ENV_FILE"
        echo "DB_PORT=\"3306\"" >> "$ENV_FILE"
        echo "DB_USER=\"<db-user>\"" >> "$ENV_FILE"
        echo "DB_PASSWORD=\"<db-user-password>\"" >> "$ENV_FILE"  # Replace with actual password
        echo "DB_NAME=\"<db-name>\"" >> "$ENV_FILE"
        
        # Install Node.js dependencies as ec2-user
        sudo -u ec2-user npm install
        
        # Start the application using PM2 as ec2-user
        sudo -u ec2-user npm run serve
        
        # Ensure PM2 restarts on reboot as ec2-user
        sudo -u ec2-user pm2 startup systemd 
        sudo -u ec2-user pm2 save
        ```
        
* Create Target Group:
    
    1. **Navigate to Target Groups**: Go to the **EC2 Dashboard** and click on **Target Groups** in the left-hand menu.
        
    2. **Create Target Group**:
        
        * **Choose Target Type**: Select **Instances**.
            
        * **Target Group Name**: Enter `application-tg`.
            
        * **Protocol**: Select **HTTP**.
            
        * **Port**: Set to **3200**.
            
        * **VPC**: Select your **created VPC**.
            
    3. **Health Check Settings**:
        
        * **Path**: Enter `/health` for the health check endpoint.
            
        * Leave other health check settings as default.
            
    4. **Create Target Group**: Click **Create Target Group** to complete the setup.
        
* Creating Application Load Balancer:
    
    1. **Navigate to Load Balancers**: Go to the **EC2 Dashboard** and select **Load Balancers**.
        
    2. **Create Load Balancer**:
        
        * **Name**: Enter `application-alb`.
            
        * **Scheme**: Select **internal**.
            
        * **Availability Zones**: Choose the **Availability Zones** and their corresponding **private subnets**.
            
    3. **Assign Security Group**:
        
        * Select the **security group** created for the **application tier ALB**.
            
    4. **Configure Routing**:
        
        * **Target Group**: Associate the ALB with the **target group** `application-tg`.
            
    5. **Create Load Balancer**: Click **Create Load Balancer** to complete the process.
        
* Create Auto Scaling Group:
    
    1. **Name the Auto Scaling Group**: Enter the name as `application-asg`.
        
    2. **Select Launch Template**: Choose the **launch template** we created earlier.
        
    3. **VPC and Subnets**: Select our VPC and the **public subnets** where the instances will be launched.
        
    4. **Attach Load Balancer**:
        
        * Choose **Attach to an existing load balancer**.
            
        * Select our **target group** `application-tg`.
            
    5. **Enable Health Checks**: Turn on **load balancer health checks**.
        
    6. **Group Metrics**: Enable **group metrics collection** in **CloudWatch**.
        
    7. **Desired Capacity**:
        
        * Set **Desired Capacity** to 3, **Minimum** 2, **Maximum** `4`.
            
    8. **Scaling Policy**:
        
        * Select **Target tracking policy**.
            
        * Set **Average CPU Utilization** to `50%`.
            

### 5\. Modify Presentation Tier:

* **Modify Launch Template**:
    
    * Edit the **user data** section in the existing **launch template**.
        
    * Add necessary commands for configuring the presentation layer (e.g., installing NGINX, pulling the React app).
        
        ```plaintext
        #!/bin/bash
        # Update package list and install required packages
        sudo yum update -y
        sudo yum install -y git
        
        # Install Node.js (use NodeSource for the latest version)
        curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
        sudo yum install -y nodejs
        
        # Install NGINX
        sudo yum install -y nginx
        
        # Start and enable NGINX
        sudo systemctl start nginx
        sudo systemctl enable nginx
        
        # Define variables
        REPO_URL="https://github.com/learnItRightWay01/react-node-mysql-app.git"
        BRANCH_NAME="feature/add-logging"
        REPO_DIR="/home/ec2-user/react-node-mysql-app/frontend"
        ENV_FILE="$REPO_DIR/.env"
        APP_TIER_ALB_URL="http://<internal-application-tier-alb-end-point.region.elb.amazonaws.com>"  # Replace with your actual alb endpoint
        API_URL="/api"
        
        # Clone the repository as ec2-user
        cd /home/ec2-user
        sudo -u ec2-user git clone $REPO_URL
        cd react-node-mysql-app
        
        # Checkout to the specific branch
        sudo -u ec2-user git checkout $BRANCH_NAME
        cd frontend
        
        # Ensure ec2-user owns the directory
        sudo chown -R ec2-user:ec2-user /home/ec2-user/react-node-mysql-app
        
        # Create .env file with the API_URL
        echo "VITE_API_URL=\"$API_URL\"" >> "$ENV_FILE"
        
        # Install Node.js dependencies as ec2-user
        sudo -u ec2-user npm install
        
        # Build the frontend application as ec2-user
        sudo -u ec2-user npm run build
        
        # Copy the build files to the NGINX directory
        sudo cp -r dist /usr/share/nginx/html/
        
        # Update NGINX configuration
        NGINX_CONF="/etc/nginx/nginx.conf"
        SERVER_NAME="<domain subdomain>"  # Replace with your actual domain name
        
        # Backup existing NGINX configuration
        sudo cp $NGINX_CONF ${NGINX_CONF}.bak
        
        # Write new NGINX configuration
        sudo tee $NGINX_CONF > /dev/null <<EOL
        user nginx;
        worker_processes auto;
        
        error_log /var/log/nginx/error.log warn;
        pid /run/nginx.pid;
        
        events {
            worker_connections 1024;
        }
        
        http {
            include /etc/nginx/mime.types;
            default_type application/octet-stream;
        
            log_format main '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                            '\$status \$body_bytes_sent "\$http_referer" '
                            '"\$http_user_agent" "\$http_x_forwarded_for"';
        
            access_log /var/log/nginx/access.log main;
        
            sendfile on;
            tcp_nopush on;
            tcp_nodelay on;
            keepalive_timeout 65;
            types_hash_max_size 2048;
        
            include /etc/nginx/conf.d/*.conf;
        }
        EOL
        
        # Create a separate NGINX configuration file
        sudo tee /etc/nginx/conf.d/presentation-tier.conf > /dev/null <<EOL
        server {
            listen 80;
            server_name $SERVER_NAME;
            root /usr/share/nginx/html/dist;
            index index.html index.htm;
        
            #health check
            location /health {
                default_type text/html;
                return 200 "<!DOCTYPE html><p>Health check endpoint</p>\n";
            }
        
            location / {
                try_files \$uri /index.html;
            }
        
            location /api/ {
                proxy_pass $APP_TIER_ALB_URL;
                proxy_set_header Host \$host;
                proxy_set_header X-Real-IP \$remote_addr;
                proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto \$scheme;
            }
        }
        EOL
        
        
        # Restart NGINX to apply the new configuration
        sudo systemctl restart nginx
        ```
        
* **Create New Version**:
    
    * Save changes and create a new version of the **launch template**.
        
* **Update Auto Scaling Group**:
    
    * Navigate to the **Auto Scaling Group**.
        
    * Edit the configuration and select the **latest version** of the launch template to ensure that the updated version is used for instance launches.
        

### 4\. CloudWatch Monitoring:

We will now monitor the application using **CloudWatch** by collecting logs from the application tier EC2 instances. The logs are located at `react-node-mysql-app/backend/logs`, and we'll move all the instance logs to **CloudWatch**. Here's a quick breakdown of the steps:

1. **Create a CloudWatch Log Group**:
    
    * Go to the **CloudWatch Console**.
        
    * Navigate to **Log Groups**, fill in the details, and create a new log group.
        
2. **Create IAM Role for EC2**:
    
    * Create a new IAM role with the following policies:
        
        * CloudWatchLogsFullAccess
            
        * CloudWatchAgentServerPolicy
            
    * Attach this IAM role to the application tier launch template.
        
    * Modify the launch template version, and under Advanced Details, select the created IAM role under the IAM profile.
        
3. **Enable Detailed CloudWatch Monitoring**:
    
    * Ensure detailed monitoring is enabled for CloudWatch.
        
4. **Add User Data Snippet**:
    
    * Update the **user data** script in the launch template with the snippet for CloudWatch Agent, configuring it to forward logs to the created log group.
        
5. **Create New Version**:
    
    * Save changes and create a new version of the **launch template**.
        
6. **Update Auto Scaling Group**:
    
    * Navigate to the **Auto Scaling Group**.
        
    * Edit the configuration and select the **latest version** of the launch template to ensure that the updated version is used for instance launches.
        

### 5\. Cloudfront and Route 53:

* Cloudfront:
    
    1. **Create CloudFront Distribution**:
        
        * Go to the **CloudFront Console**.
            
        * Click on **Create Distribution**.
            
    2. **Configure Origin**:
        
        * **Origin Domain Name**: Select the domain name of your presentation tier ALB.
            
        * **Origin Protocol Policy**: Choose **HTTP Only** to ensure communication is established over HTTP.
            
    3. **Viewer Protocol Policy**:
        
        * Set the **Viewer Protocol Policy** to **Redirect HTTP to HTTPS**. This will ensure that all HTTP requests are redirected to HTTPS for better security.
            
    4. **Web Application Firewall (WAF)**:
        
        * Disable WAF by not associating it with the distribution for now.
            
    5. **Alternate Domain Names**:
        
        * In the **Alternate Domain Names (CNAMEs)** field, insert any custom domain names you wish to use for the distribution.
            
    6. **Create Distribution**:
        
        * Review your configurations and click on **Create Distribution** to finalize the setup.
            
* Route 53:
    
    1. **Create Record**:
        
        * In the **Hosted Zones** page, select the hosted zone you just created.
            
        * Click on **Create record**.
            
    2. **Configure Record Settings**:
        
        * **Record name**: Leave this field empty for the root domain or enter a subdomain (e.g., `www`).
            
        * **Record type**: Choose **A – IPv4 address**.
            
        * **Alias**: Select **Yes**.
            
        * **Alias Target**: In the dropdown, find and select your CloudFront distribution from the list.
            
    3. **Routing Policy**:
        
        * Choose the routing policy (Simple Routing is usually appropriate).
            
    4. **TTL (Time to Live)**:
        
        * Set the TTL as desired (default is typically 300 seconds).
            
    5. **Create Record**:
        
        * Click on **Create records** to finalize the setup.
            
    6. Ready to access you three tier web application.
        

### Conclusion

In this blog, we successfully deployed a robust three-tier application architecture on AWS, leveraging services like VPC, EC2, RDS, and CloudFront to ensure high availability, security, and performance. The integration of Route 53 for domain management, including creating a hosted zone and alias records for our CloudFront distribution, enables seamless access to our application.

By implementing Auto Scaling and Load Balancing, we created a flexible and scalable solution capable of handling varying traffic loads. Additionally, CloudWatch monitoring allows us to track application performance and maintain operational health effectively. The inclusion of a bastion host enhances security by limiting direct access to our resources.

This deployment strategy, along with the best practices discussed, provides a strong foundation for building and managing production-ready applications in the cloud.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
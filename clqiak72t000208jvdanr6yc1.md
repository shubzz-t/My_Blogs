---
title: "Docker Multi-container Communication"
seoTitle: "Multi Container comm"
datePublished: Sat Dec 23 2023 16:45:16 GMT+0000 (Coordinated Universal Time)
cuid: clqiak72t000208jvdanr6yc1
slug: docker-multi-container-communication
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701111172425/26820866-6e2b-4186-8a64-96c5827b35e7.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1703349876650/1a364035-cdf0-4738-9526-3fa88a15d08f.jpeg
tags: docker, devops, docker-network, trainwithshubham

---

Ever wondered how multiple containers in Docker talk to each other? Our blog has got you covered! We'll break down the ins and outs of communication between containers, offering practical tips and strategies. Whether you're new to Docker or a pro, this guide simplifies the complexities, making it easy to establish smooth connections between your Docker containers. Get ready to boost your Docker skills and enhance your multi-container setups with our straightforward insights.

To kick start our exploration of Docker multi-container setups, the first step is to clone the code from our GitHub repository into your system.

[Github Repository URL :- https://github.com/shubzz-t/flask\_app\_two\_tier](https://github.com/shubzz-t/flask_app_two_tier)

This code base encapsulates a two-tier application crafted with Flask for the front end and MySQL serving as the database back-end. Our goal is to containerise this application effectively, and for that, we'll be creating two containers. The first container will host the Flask application, responsible for the front-end logic, while the second container will house the MySQL database, forming the backbone of our application. The magic happens when we connect these two containers, paving the way for seamless communication between the Flask web app and the MySQL database.

We'll tackle the challenge from two angles:

### 1 ) Firstly, by dividing the process using Dockerfiles :

* **Create Bridge network :**
    
    Create a bridge network that will ensure both containers reside on the same network. This bridge network acts as a virtual environment, allowing containers to communicate effortlessly while maintaining isolation from the host system.
    
    ```bash
    #Command to create network where network name is flask_app_nw
    docker create -d bridge flask_app_nw
    ```
    
* **Pull and Run MySQL image :**
    
    Once our bridge network, named 'flask\_app\_nw,' is in place, the next step is to pull and run the MySQL image, ensuring that it's part of our newly created network. By explicitly specifying our bridge network during the pull and run processes, we're ensuring that the MySQL container is seamlessly integrated into our Docker network environment. In addition to linking it to the network, it's crucial to specify the appropriate environment variables required for the MySQL image.
    
    ```bash
    #Command to run mysql container with network bridge and setting the environment variables for database
    docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=test@123 -e MYSQL_DATABASE=testdb -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin --name mysql --network flask_app_nw  mysql:latest
    ```
    
* **Writing Dockerfile for creating flask image:**
    
    The next crucial step is to create the Flask image from the cloned code. Leveraging the Dockerfile, we'll encapsulate the necessary configurations and dependencies to ensure a self-contained and reproducible image.
    
    As we have cloned the repository we will be having the folder flask\_app\_two\_tier go inside it and create Dockerfile.
    
    Dockerfile
    
    ```yaml
    # Use an official Python runtime as the base image
    FROM python:3.9-slim
    
    # Set the working directory in the container
    WORKDIR /app
    
    # install required packages for system
    RUN apt-get update \
        && apt-get upgrade -y \
        && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
        && rm -rf /var/lib/apt/lists/*
    
    # Copy the requirements file into the container
    COPY requirements.txt .
    
    # Install app dependencies
    RUN pip install mysqlclient
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Copy the rest of the application code
    COPY . .
    
    # Specify the command to run your application
    CMD ["python", "app.py"]
    ```
    
* **Build the flask image from the Dockerfile:**
    
    Here, we will build the flask\_app whose Dockerfile is present in current directory.
    
    ```bash
    #Command to create flask container from Dockerfile
    docker build --t flask_app .
    ```
    
* **Run the flask container from image:**
    
    With our Flask image now ready, it's time to take the next leap and run the image, transforming it into a fully functional container. During this process, we'll strategically link our Flask container to the previously created bridge network, 'flask\_app\_nw.' Additionally, we'll set the essential environment variables required for our Flask image to connect with the MySQL image. These variables, encompassing details such as MySQL user, password, and database name, establish the necessary configuration for the Flask container to interact with its MySQL counterpart.
    
    ```bash
    #Command to run the container with attached bridge network and required environment variables for connection
    docker run -d -p 5000:5000 -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_DB=testdb --name flask_app --network flask_app_nw flask_app:latest
    ```
    

### 2 ) Secondly exploring the streamlined orchestration capabilities of Docker Compose.

Docker Compose streamlines the orchestration of multi-container applications by consolidating configuration details into a single YAML file, offering a more efficient alternative to the manual setup of individual Docker containers. Docker Compose simplifies the process by allowing developers to define the entire application stack, including services, networks, and volumes, in a centralized configuration. Docker Compose facilitates seamless communication between containers, while also providing a straightforward mechanism for managing data persistence through defined volumes.

* **Creating a docker-compose.yml file:**
    
    Firstly we need to create a file with the name docker-compose.yml this file name specification is mandatory.
    
    ```bash
    vi docker-compose.yml
    ```
    
* **Writing the docker-compose.yml file:**
    
    Certainly! Below is a sample Docker Compose file that includes configuration details for both a front-end and a back-end container. Adjust the image names, ports, environment variables, and other settings based on your specific requirements.
    
    ```yaml
    version: '3'
    services:
      
      backend:
        build:
          context: .
        ports:
          - "5000:5000"
        environment:
          MYSQL_HOST: mysql
          MYSQL_USER: admin
          MYSQL_PASSWORD: admin
          MYSQL_DB: myDb
        depends_on:
          - mysql
    
      mysql:
        image: mysql:5.7
        ports:
          - "3306:3306"
        environment:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: myDb
          MYSQL_USER: admin
          MYSQL_PASSWORD: admin
        volumes:
          - ./message.sql:/docker-entrypoint-initdb.d/message.sql   # Mount sql script into container's /docker-entrypoint-initdb.d directory to get table automatically created
          - mysql-data:/var/lib/mysql  # Mount the volume for MySQL data storage
    
    volumes:
      mysql-data:
    ```
    
    This above file will create 2 containers flask and MySQL along with network bridge.
    
* **Running the docker-compose file:**
    
    Using the below command you can run the docker-compose file in order to check the flask app is up with the database.
    
    ```bash
    #docker-compose up command to run containers and -d for deamon mode
    sudo docker-compose up -d
    ```
    
    After this you can check the working application is up on your instance ip:5000 port.
    

And that's a wrap! So, we've uncovered the secrets of how Docker containers chat with each other. Whether you're a Docker rookie or a seasoned pro, our guide has laid it all out for you. Now, connecting your containers is a breeze with the tips and tricks we've shared.

If you have any questions, need further clarification, or just want to share your Docker success stories, feel free to reach out. I'm here to help and eager to hear about your containerized adventures. Drop me a message, and let's keep the conversation going. Happy coding!
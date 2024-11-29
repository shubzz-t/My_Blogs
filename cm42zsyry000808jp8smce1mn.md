---
title: "Docker Builds with Multi-Architecture Support"
datePublished: Fri Nov 29 2024 17:03:29 GMT+0000 (Coordinated Universal Time)
cuid: cm42zsyry000808jp8smce1mn
slug: docker-builds-with-multi-architecture-support
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732788335854/2ef14e3b-c35c-4c92-b4de-d01c9f1bfdac.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1732899796290/4e3b40b9-2c57-44ad-973f-649337bd9fbe.png
tags: docker, aws, architecture, devops, multiarch

---

In the world of software development, Docker has become a go-to tool for creating, deploying, and running applications inside containers. However, a common challenge developers faced was building containers that could run on different types of hardware, like ARM-based systems (e.g., Raspberry Pi) and x86-based systems (e.g., most PCs and servers). Before Docker introduced multi-architecture builds, developers had to manually create different versions of their containers for each platform, which was time-consuming and prone to errors.

With Docker's multi-architecture support, this problem is now solved. Developers can build and push images that work across multiple platforms, all from a single Dockerfile. In this blog, we’ll explore why multi-architecture builds are essential, how Docker’s new features solve past issues, and how you can use them to streamline your containerization workflow.

### Issue before multi architecture builds:

Before Docker introduced multi-architecture builds, the main issue was that developers had to create separate Docker images for different types of hardware platforms, like x86 (used in most desktops and servers) and ARM (used in devices like Raspberry Pi).

* **Manual effort**: Developers had to build, test, and maintain different Docker images for each platform manually.
    
* **Time-consuming**: It took a lot of extra time and effort to ensure that the images worked on all platforms.
    
* **Error-prone**: With multiple versions of the same app for different platforms, mistakes could happen, like forgetting to update one version or introducing bugs in some platforms.
    
* **Compatibility issues**: Some software or libraries worked well on one architecture but not on another, leading to more problems when trying to run the app on different systems.
    

### Issues resolved using Multi architecture builds:

1. **Single Image for All Platforms**: Instead of creating separate images for each platform (e.g., ARM, x86), Docker now lets you build one image that works on multiple architectures.
    
2. **Automated Builds**: Docker can automatically build images for different platforms at once, saving developers time and effort.
    
3. **Easier Distribution**: When you push an image to Docker Hub, users automatically get the right version for their system, whether it’s ARM or x86.
    
4. **Consistency Across Platforms**: The same app works consistently across different devices, like servers and Raspberry Pis, without compatibility issues.
    
5. **Simplified Workflow**: Developers no longer need to manually switch between machines or deal with multiple Dockerfiles, making the process faster and less error-prone.
    

### Creating Multi Arch Image:

To create multi-architecture Docker images, you need to use Docker's **Buildx** tool, which is an extension of the standard `docker build` command.

**PREREQUISITES:**

1. **Docker Desktop** (Windows or MAC) / **Docker Engine** for Linux
    
2. **Enable Experimental Features (Windows)**  
    Docker Buildx is an experimental feature, so you need to enable it in Docker Desktop (if you're using Docker Desktop). Go to **Settings** &gt; **Docker Engine**, and in the JSON config, add `"experimental": true`, then restart Docker.
    
3. **Docker Buildx Plugin**  
    Docker Buildx is included with Docker Desktop by default, but on Linux, you might need to install it manually. It’s a command-line plugin that extends the `docker build` command to support multi-platform builds.
    
    ```plaintext
    sudo apt install docker-buildx
    ```
    
    Verify installation using command:
    
    ```plaintext
    docker builder ls
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732783527812/855541ba-f979-4f75-b89a-b745ff1c83d1.png align="center")
    

**CREATING THE MULTI ARCH IMAGE USING LINUX:**

* **Set Up Buildx**
    
    First, ensure you're using Buildx by creating and setting up a new builder instance.
    
    * Run the following command to create and set the builder:
        
        ```plaintext
        sudo docker buildx create --name multiarch --platform linux/amd64,linux/arm64 --driver docker-container --bootstrap --use
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732784295288/8d9e06a6-2825-4d89-aba1-d9e6e4a63657.png align="center")
        
    * Verify builder creation using command:
        
        ```plaintext
        sudo docker builder ls
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732784376861/c422d3a3-4aac-4dc3-b919-7eaab790f58d.png align="center")
        
* Now, you can build your Docker image for multiple architectures. Use the `docker buildx build` command with the `--platform` flag to specify the architectures.
    
    Here's an example where you build an image for both `x86` and `ARM64`:
    
    ```plaintext
    docker buildx build --platform linux/amd64,linux/arm64 -t shubzz/multiarch:latest .
    ```
    
    * `--platform linux/amd64,linux/arm64`: Specifies the target platforms (you can add more platforms if needed).
        
    * `-t myimage:latest`: Tags the image as `myimage:latest`.
        
    * `.`: Refers to the current directory, where your `Dockerfile` is located.
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732784476981/30abf4e2-1058-485b-b341-fa681bbfb47b.png align="center")
    
* We can verify our image in Dockerhub with multi architecture types. We can see image is persent in 2 architecture types.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732788035372/a15eefa3-2e75-4e5e-9314-1d91a17adff5.png align="center")
    

### **How Docker Handles Multi-Architecture Images**

1. **Manifest List**: When you push a multi-architecture image (using `docker buildx build` with the `--platform` flag), Docker creates a **manifest list**. A manifest list is a collection of individual manifests, where each manifest corresponds to a specific platform/architecture (e.g., `linux/amd64`, `linux/arm64`, etc.).
    
2. **Automatic Architecture Selection**: When you (or your system) pull a multi-architecture image, Docker automatically detects the architecture of the system (whether it's `amd64`, `arm64`, or another supported architecture). Based on this, Docker will pull the correct variant of the image that matches the system's architecture.
    
3. **Example Flow**:
    
    * You build and push an image to Docker Hub with the following command:
        
        ```plaintext
        docker buildx build --platform linux/amd64,linux/arm64 -t shubzz/multiarch:latest --push .
        ```
        
        This will create a multi-architecture image that can be used on both `amd64` and `arm64` systems.
        
    * If a system with `amd64` (x86\_64) architecture pulls the image:
        
        ```plaintext
        docker pull shubzz/multiarch:latest
        ```
        
        Docker will pull the `linux/amd64` version of the image.
        
    * If a system with `arm64` (e.g., an ARM-based server or Raspberry Pi) pulls the same image:
        
        ```plaintext
        docker pull shubzz/multiarch:latest
        ```
        
        Docker will pull the `linux/arm64` version of the image.
        

### Conclusion:

In summary, Docker's multi-architecture builds solve the problem of supporting different system architectures (like `amd64` and `arm64`) with a single image. Instead of building separate images for each platform, you can now create one image that works across multiple architectures. Docker automatically selects the right version of the image based on the system’s architecture, making it easier to manage and deploy applications on different devices. By using tools like `docker buildx` and the `--platform` flag, you can build, push, and pull images that support various architectures seamlessly, ensuring your applications work everywhere—from x86 servers to ARM-based devices like Raspberry Pi.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
---
title: "Automating AWS Infrastructure Provisioning with Terraform and GitLab CI"
datePublished: Mon Sep 02 2024 06:28:27 GMT+0000 (Coordinated Universal Time)
cuid: cm0kmdcfm000i09mebv87g2w9
slug: automating-aws-infrastructure-provisioning-with-terraform-and-gitlab-ci
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725227988582/5e22df1d-87c7-4fd1-9644-079c33a7a906.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1725258464115/82aa292d-9f82-4ce9-a8a8-2cf4810cb3de.png
tags: aws, automation, devops, gitlab, terraform, vpc

---

In today's cloud-driven world, managing infrastructure efficiently is essential. This project focuses on using Terraform to set up and manage AWS resources. We use Amazon S3 to store the Terraform state file, keeping track of our infrastructure's current state securely.

The highlight of this project is the automation: by integrating GitLab CI/CD, the entire process of setting up infrastructure is automated, so no manual work is needed. This makes deploying resources faster, more reliable, and easier to manage.

**PREREQUISITES:**

* AWS Account
    
* Gitlab Account
    
* Machine with terraform and AWS CLI installed
    
* AWS Access Keys
    

### Step 1: Writing terraform code for VPC and EC2:

We’ll organize our Terraform code using modules. We’ll create a **VPC module** to set up networking components and a **Web module** to provision EC2 instances for our web servers. These modules help keep our code clean, reusable, and scalable.

* Following will be our file structure:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725222381868/e6eeeb7b-fe45-4137-9974-59e8d6282e5a.png align="center")
    
* You can get the above code here: [https://gitlab.com/basics4530332/terraform-cicd.git](https://gitlab.com/basics4530332/terraform-cicd.git)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725226774436/5f7d96db-10bb-422b-8196-318bf065aaca.png align="center")
    
* In the above code you need to change the backend.tf with your created bucket name and your created DynamoDB table.
    

### Step 2: Setting up the pipeline to provision infrastructure:

Here we will write the pipeline code to validate, plan, apply and destroy the AWS infrastructure.

* Creating the variables for the access keys:
    
    1. In your GitLab repository go to &gt; Settings &gt; CI/CD &gt; select variables &gt; Create variables for AWS access and secret access key.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725225948906/b09383fe-6e87-4d01-8f88-e33dc85189a1.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725225984637/d08c662e-d5a8-4340-a16e-9d25e058d1ce.png align="center")
        
* Now we will be writing pipeline script for that create a file with the filename as .gitlab-ci.yml. Following will be the pipeline script.
    
    ```yaml
    image:
      name: registry.gitlab.com/gitlab-org/gitlab-build-images:terraform
      entrypoint:
        - '/usr/bin/env'
        - 'PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
    
    variables:
      AWS_ACCESS_KEY_ID: ${MY_AWS_KEY}
      AWS_SECRET_ACCESS_KEY : ${MY_AWS_ACCESS_KEY}
      AWS_DEFAULT_REGION: "ap-south-1"
    
    cache:
      paths:
        - .terraform
    
    before_script:
      - terraform --version
      - terraform init 
    
    stages:
      - validate
      - plan
      - apply
      - destroy
    
    validate:
      stage: validate
      script:
        - terraform validate
    
    plan:
      stage: plan
      script:
        - terraform plan -out="planfile"
      dependencies:
        - validate
      artifacts:
        paths:
          - planfile
    
    apply:
      stage: apply
      script:
        - terraform apply -input=false "planfile"
      dependencies:
        - plan
      when: manual
    
    destroy: 
      stage: destroy
      script:
        - terraform destroy --auto-approve
      when: manual 
    ```
    
    1. **Image:** Uses a Docker image with Terraform pre-installed to run the pipeline.
        
    2. **Variables:** Sets up AWS credentials and the default region using environment variables (`MY_AWS_KEY`, `MY_AWS_ACCESS_KEY`).
        
    3. **Cache:** Caches the `.terraform` directory to speed up future pipeline runs.
        
    4. **Before Script:** Ensures Terraform is initialized and ready by checking its version and running `terraform init`.
        
    5. **Stages:** Defines four stages: `validate`, `plan`, `apply`, and `destroy`.
        
        * **Validate:** Checks the Terraform configuration for syntax errors using `terraform validate`.
            
        * **Plan:** Generates an execution plan (`terraform plan`) and saves it as `planfile`. The `dependencies` keyword ensures this stage only runs after the `validate` stage. The `artifacts` section stores the `planfile` for use in later stages.
            
        * **Apply:** Applies the saved plan file to provision resources on AWS (`terraform apply`). The `dependencies` keyword makes sure this stage only runs after the `plan` stage. The `when: manual` keyword ensures that the apply stage requires manual approval before it executes, providing control over when changes are applied.
            
        * **Destroy:** This stage destroys the AWS infrastructure (`terraform destroy`) without needing confirmation (`--auto-approve`). It’s also set to run manually with `when: manual`, allowing you to destroy the infrastructure only when you decide to.
            
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725226543656/e45879ef-1141-49b8-b275-07fba3975d21.png align="center")
    

### Conclusion:

This project shows how to use Terraform with GitLab CI/CD to automate AWS infrastructure management. By structuring our Terraform code with modules and automating the pipeline, we streamline deployments and reduce manual tasks. Manual approvals for critical stages ensure control and safety in our automation process, making it efficient and reliable.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

**Streamline, Deploy, Succeed-- Devops Made Simple!☺️**
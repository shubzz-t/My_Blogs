---
title: "How to Grant AWS Users Access to an EKS Cluster: A Step-by-Step Guide"
datePublished: Sat Nov 23 2024 10:00:24 GMT+0000 (Coordinated Universal Time)
cuid: cm3u01rp900040akz99rq5myl
slug: how-to-grant-aws-users-access-to-an-eks-cluster-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732355665605/d52940aa-7508-409a-9c48-afecdc695267.gif
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1732355967379/c6aae9c7-7cb4-4867-8fa7-e5182167a1db.gif
tags: authentication, aws, kubernetes, iam

---

Managing user access to resources in Amazon EKS is crucial for security and control. In this blog, we'll show you how to give a new user, Shubham, **read-only access** to an EKS cluster. This is especially useful when you want someone to monitor Kubernetes resources without the risk of accidental changes or deletions.

We’ll walk through the entire process—from setting up an IAM user, creating the necessary policies, to configuring the `kubeconfig` file for Shubham to access the cluster. By the end, you’ll be able to easily provide read-only access to any user in your EKS environment. Let’s get started!

### **Step 1: Create IAM User for Shubham**

1. **Go to the IAM Console**: AWS IAM Console
    
2. **Create a new IAM user**:
    
    * Click **Users** → **Add user**.
        
    * Enter **User name** (e.g.,`shubham`).
        
    * Select **Access type**:
        
        * **Programmatic access**: for AWS CLI.
            
        * **AWS Management Console access**: if needed for the console (optional).
            
    * Click **Next: Permissions**.
        
3. **Attach Permissions**: To restrict Shubham to read-only access for EKS resources, **do not grant** full access to EKS resources. Instead, we will create a custom IAM policy for Shubham.
    
    * Click **Attach policies directly**.
        
    * **Search policies** according to the access you want to give here I want read-only access so I have used custom policy.
        
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "eks:DescribeCluster",
                        "eks:ListClusters",
                        "eks:DescribeNodegroup",
                        "eks:ListNodegroups",
                        "eks:DescribeUpdate",
                        "eks:ListUpdates",
                        "eks:DescribeFargateProfile",
                        "eks:ListFargateProfiles",
                        "eks:DescribeAddon",
                        "eks:ListAddons"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:DescribeInstances",
                        "ec2:DescribeSecurityGroups",
                        "ec2:DescribeSubnets",
                        "ec2:DescribeVpcs",
                        "ec2:DescribeTags"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "cloudwatch:DescribeAlarms",
                        "cloudwatch:ListMetrics",
                        "cloudwatch:GetMetricData",
                        "cloudwatch:GetDashboard"
                    ],
                    "Resource": "*"
                }
            ]
        }
        ```
        
4. Click **Next: Tags** → **Next: Review** → **Create user**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352849541/437b9db0-b01f-4aa1-a8d4-9f5031429880.png align="center")
    
5. **Download Shubham's credentials** (Access Key ID and Secret Access Key) securely.
    

### **Step 2: Add IAM User to the EKS Cluster’s** `aws-auth` ConfigMap

To ensure Shubham can interact with the EKS cluster, you must add his IAM user to the EKS cluster's `aws-auth` configmap.

#### **2.1. Update** `aws-auth` ConfigMap

1. **Ensure you have kubectl access to the EKS cluster**. If not, follow the steps to generate and configure your `kubeconfig` (covered below).
    
2. **Edit the** `aws-auth` configmap: Run the following command to edit the `aws-auth` configmap:
    
    ```plaintext
    kubectl edit configmap aws-auth -n kube-system
    ```
    
3. **Add Shubham’s IAM user to the** `mapUsers` section: In the `aws-auth` configmap, add Shubham’s IAM user under the `mapUsers` section:
    
    ```plaintext
    apiVersion: v1
    data:
      mapUsers: |
        - userarn: arn:aws:iam::ACCOUNT-ID:user/shubham
          username: shubham
          groups:
            - system:masters
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352930797/3087ea61-b860-4aef-9a46-1787c95bda1d.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352948667/bd5031f5-7e8c-467b-a961-d67861b0293f.png align="center")
    
    **Important:** You want to **replace** `system:masters` with a **read-only group** in Kubernetes.
    
    If you want **read-only access**, we'll use a custom Kubernetes role binding (explained below). So you don't need to assign Shubham to `system:masters`.
    
    Save and exit the editor.
    

### **Step 3: Create Read-Only Kubernetes Role**

To give Shubham **read-only access** to the Kubernetes cluster, you need to create a **ClusterRole** with read-only permissions and bind it to Shubham's Kubernetes user.

#### **3.1. Create a Read-Only ClusterRole**

Create a file called `read-only-clusterrole.yaml` with the following content:

```plaintext
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-only
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "namespaces", "deployments"]
    verbs: ["get", "list"]
  - apiGroups: ["apps"]
    resources: ["deployments", "statefulsets", "replicasets"]
    verbs: ["get", "list"]
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352985411/cad9fb02-9467-49ba-a3c6-889668d94fdf.png align="center")

This role allows **viewing** Pods, Services, Deployments, and other resources, but does not allow modifications.

#### **3.2. Create ClusterRoleBinding:**

Next, create a binding for Shubham that grants him this read-only access. Create a file called `read-only-binding.yaml` with the following content:

```plaintext
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: shubham-read-only-binding
subjects:
  - kind: User
    name: "shubham"  # Use the IAM username or Kubernetes user name
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: read-only
  apiGroup: rbac.authorization.k8s.io
```

This binds Shubham’s IAM user to the `read-only` ClusterRole.

#### **3.3. Apply the ClusterRole and ClusterRoleBinding**

To apply the `ClusterRole` and `ClusterRoleBinding`, run:

```plaintext
kubectl apply -f read-only-clusterrole.yaml
kubectl apply -f read-only-binding.yaml
```

### **Step 4: Generate** `kubeconfig` for Shubham

Now we need to generate the `kubeconfig` file, which Shubham will use to interact with the EKS cluster using `kubectl`.

#### **4.1. Generate** `kubeconfig`

Run the following command to generate the `kubeconfig` file:

```plaintext
aws eks --region ap-south-1 update-kubeconfig --name my-cluster --region <region> update-kubeconfig --name <cluster-name>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732351930137/c6851600-a2f7-40fb-84bc-f0ed520c2b54.png align="center")

* This will configure Shubham’s `~/.kube/config` file (on his machine) to access the cluster.
    
* Sharing the file with Shubham, simply give him the `~/.kube/config` file.
    

**4.2 Configuration on Shubham machine:**

Place the kubeconfig file in the right locationPlace the kubeconfig file in the right location:

* After you send Shubham the `kubeconfig` file, he will need to place it in the correct location on his machine. By default, the `kubeconfig` file is stored at `~/.kube/config`.
    

* **Copy the kubeconfig** file to `~/.kube/config`:
    
    ```plaintext
    dir -p ~/.kube
    cp /path/to/kubeconfig ~/.kube/config
    ```
    
* If there’s an existing `kubeconfig` file, he can either **merge** the new file or **replace** the old one.
    

### **Step 5: Install AWS CLI and kubectl on Shubham’s Machine**

Shubham will need the **AWS CLI** and **kubectl** installed on his machine to interact with AWS and the EKS cluster.

#### **5.1. Install AWS CLI**

1. On **Ubuntu**, run:
    
    ```plaintext
    sudo apt-get update
    sudo apt-get install awscli
    ```
    
2. Verify the installation:
    
    ```plaintext
    aws --version
    ```
    

#### **5.2. Configure AWS CLI with Shubham’s Credentials**

Shubham will now configure the AWS CLI using the Access Key ID and Secret Access Key that you provided.

1. Run the following command:
    
    ```plaintext
    aws configure
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352015549/4855dcf4-df75-4b12-9400-7f5d87c1fa65.png align="center")
    

#### **5.3. Install kubectl**

Shubham will need `kubectl` to interact with the EKS cluster.

1. Run the following commands to install `kubectl`:
    
    ```plaintext
    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```
    
2. Verify `kubectl` installation:
    
    ```plaintext
    kubectl version --client
    ```
    

---

### **Step 6: Verifying Access**

Shubham can now verify his read-only access to the EKS cluster using `kubectl`.

#### **6.1. Verify Cluster Access**

Shubham can run the following command to check if he can access the clust[er:](https://console.aws.amazon.com/iam/)

```plaintext
kubectl get nodes
```

This should list the worker nodes if Shubham has access.

#### **6.2. Verify Read-Only Permissions**

Shubham can also run the following commands to ensure he has read-only access:

```plaintext
kubectl get all -n kube-system
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352283762/0191655d-bfc8-4b8f-a2d3-b655304ef5b0.png align="center")

Shubham should not be able to modify or delete anything, but he can **list** and **view** resources.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732352332609/079152aa-cd61-4c10-aef1-7cdfc167cdbb.png align="center")

Above we can say that if we try to apply yml if gives error that shubham cannot modify as he is not having permission.

### **Conclusion:**

In this guide, we've shown how to grant AWS users access to an EKS cluster using IAM roles and Kubernetes role bindings. By following these steps, you can securely manage user access and ensure the right permissions for your team. Whether for full or limited access, this process helps maintain a secure and controlled EKS environment.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
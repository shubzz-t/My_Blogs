---
title: "Backing Up and Restoring Etcd with etctctl and Velero in Kubernetes: Key Insights for CKA and Real-World Applications""
datePublished: Sat Nov 09 2024 11:29:04 GMT+0000 (Coordinated Universal Time)
cuid: cm3a31unr000008jwag4s82pb
slug: backing-up-and-restoring-etcd-with-etctctl-and-velero-in-kubernetes-key-insights-for-cka-and-real-world-applications
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1731151619749/5da2c1c4-fd28-4e6c-b6c2-478449320cad.gif
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1731151728697/cb413bf6-fac3-4c2c-a13c-519658a145c2.gif
tags: aws, kubernetes, devops, eks, disaster-recovery, velero, etcd, kubeadm

---

In Kubernetes, data consistency and disaster recovery are critical to ensuring the availability of applications and services. The **etcd** database serves as the heart of Kubernetes, storing the cluster’s state and configurations, from deployments to secrets and persistent volumes. However, like any database, it can become vulnerable to failure, accidental deletions, or even corruption.

In this blog, we will walk you through the essential steps for **backing up and restoring the etcd database** in a **self-managed Kubernetes cluster** created with `kubeadm`. We’ll also dive deep into **Velero**, a powerful tool for backing up **Kubernetes resources** and **persistent volumes**. You’ll gain a clear understanding of how Velero complements `etcd` backups, what makes them different, and why you need both in your disaster recovery strategy.

As one of the most important interview topics and a highly relevant question for the **Certified Kubernetes Administrator (CKA)** exam, mastering **etcd backup and restore** is crucial for any Kubernetes professional. By the end of this blog, you’ll have practical knowledge of **how to perform etcd backups**, **restore operations**, and utilize **Velero for Kubernetes resource backup**—equipping you with the skills needed to confidently handle these tasks in production environments.

Let’s dive into the details of backing up your Kubernetes state and ensuring you have a robust disaster recovery plan in place!

### 1\. Backup/Restore using ETCDCTL:

In this section, we will walk through the process of backing up and restoring the **etcd** state in a **self-managed Kubernetes cluster** (for example, a cluster deployed using `kubeadm`) using `etcdctl`.

* Prerequisites:
    
    1. You are running a **self-managed Kubernetes cluster** (e.g., a cluster set up using `kubeadm`).
        
    2. You have access to the **master node** or wherever etcd is running in your Kubernetes cluster.
        
    3. You have `etcdctl` installed. This is the command-line tool to interact with **etcd**.
        
    4. You have access to the **etcd API** and the necessary credentials (if any).
        
* **Getting Control Plane Data:**
    
    1. The Kubernetes control plane components, are defined as static Pods in YAML files located in the `/etc/kubernetes/manifests/` directory. To view **etcd** configuration, you can check the specific manifest file for the **etcd** Pod i.e. etcd.yaml
        
    2. Copy the below details in notepad from your etcd.yaml.
        
        ```plaintext
        --listen-client-urls=https://127.0.0.1:2379
        --cert-file=/etc/kubernetes/pki/etcd/server.crt
        --key-file=/etc/kubernetes/pki/etcd/server.key
        --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
        ```
        
* Set Environment Variables for `etcdctl` to avoid confusion
    
    ```plaintext
    export ETCDCTL_API=3
    export ETCDCTL_CERT_FILE=/etc/kubernetes/pki/etcd/server.crt
    export ETCDCTL_KEY_FILE=/etc/kubernetes/pki/etcd/server.key
    export ETCDCTL_CA_FILE=/etc/kubernetes/pki/etcd/ca.crt
    export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379
    
    #OR
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
      --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
      snapshot save <backup-file-location>
    ```
    
* Take a Backup of the Etcd Server:
    
    ```plaintext
    etcdctl snapshot save /path/to/backup/etcd-backup.db
    ```
    
* Verify the backup using command:
    
    ```plaintext
    etcdctl snapshot status /path/to/backup/etcd-backup.db
    ```
    
* Restore etcd if there is any issue in cluster:
    
    1. If any API servers are running in your cluster, you should not attempt to restore instances of etcd. Instead, follow these steps to restore etcd:
        
        * **stop *all* API server instances**
            
            just move the manifests out of the `/etc/kubernetes/manifests` dir and kubelet will stop the containers gracefully.
            
            ```plaintext
            sudo systemctl stop etcd
            sudo systemctl start  etcd
            ```
            
        * **restore state in all etcd instances**
            
        * **restart all API server instances**
            
            just move the manifests into the `/etc/kubernetes/manifests` dir and kubelet will start the containers gracefully.
            
    2. Restoring the snapshot:
        
        ```plaintext
        etcdutl --endpoints=https://127.0.0.1:2379 \
          --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
          --data-dir <data-dir-location> snapshot restore snapshot.db
        
        #OR
        etcdutl --data-dir <new-dir-location> snapshot restore snapshot.db
        ```
        
    3. Changing the data directory in /etc/kubernetes/manifests/etcd.yaml:
        
        * After restoring the snapshot, you need to update the `etcd` configuration to use the new data directory (`/var/lib/etcd-new`) where the restored data has been placed.
            
        * Look for the --data-dir flag and change it to your new data directory:
            
            ```plaintext
            --data-dir=/new/data/dir/etcd-new
            ```
            
    4. You’ll need to update the `volumeMounts` and `volumes` sections in the `etcd.yaml` file to reflect the new `data-dir` and volume paths.
        
    5. sudo systemctl daemon-reload
        
    6. sudo systemctl restart kubelet
        
    7. check using the command kubectl describe pod etcd-master -n=kube-system
        
* **etcdctl Advantages:**
    
    1. **Direct Access to etcd**: Provides direct access to backup and restore the Kubernetes control plane data stored in etcd.
        
    2. **Simple and Lightweight**: Minimalistic tool focused solely on etcd, making it lightweight for control plane backup.
        
    3. **Fine-Grained Control**: Offers granular control over the etcd backup process (manual snapshots, incremental backups).
        
    4. **Self-Managed Clusters**: Ideal for self-managed Kubernetes clusters where you control the etcd instance.
        
* **etcdctl Disadvantages:**
    
    1. **Limited to etcd**: Only backs up **etcd data**; does not cover workloads, services, or persistent volumes.
        
    2. **Manual Operation**: Requires manual snapshots and interventions, making it less automated than other solutions.
        
    3. **No Cloud Integration**: Lacks native support for cloud storage or cross-region backups.
        
    4. **Not Suitable for Managed Clusters**: Cannot be used in cloud-managed Kubernetes (like EKS, AKS) since you don't have direct access to etcd.
        
    5. **No Namespace/Resource-Level Backups**: Cannot perform backups for specific namespaces or other Kubernetes resources beyond etcd.
        

### What to use for Cloud Managed Cluster?

**Velero**. Here's why:

1. **Cloud Provider Integration**: Velero integrates with cloud storage services like **S3 (AWS)**, **Google Cloud Storage**, and **Azure Blob Storage**, which are ideal for storing backup data in cloud-managed environments.
    
2. **Full Kubernetes Backup**: Velero can back up not just the **etcd** data (control plane), but also **Kubernetes resources** (pods, services, deployments, etc.), **persistent volumes**, and **cloud-specific resources**.
    
3. **Automated and Scheduled Backups**: Velero supports automated backups, including scheduled and incremental backups, reducing manual intervention.
    
4. **Disaster Recovery**: It supports **cross-region** and **multi-cluster** restores, which are essential for cloud-managed clusters with global availability.
    
5. **Namespace-Level Backups**: Velero allows for **namespace-level** backup and restore, making it more flexible for selective restores.
    
6. **No Direct etcd Access**: Since cloud-managed clusters abstract the etcd database, you cannot directly interact with etcd (like with `etcdctl`), making Velero the ideal solution for full cluster backup and restoration.
    

### 2\. Backup/Restore using Velero:

* **Step 1: Velero CLI Installation**
    
    To begin, you need to install the **Velero CLI** on your local machine or wherever you will manage the backups from.
    
    1. **Download the Velero CLI**:
        
        ```plaintext
        wget https://github.com/vmware-tanzu/velero/releases/download/v1.7.0/velero-v1.7.0-linux-amd64.tar.gz
        ```
        
        This command downloads the Velero CLI release tarball.
        
    2. **Extract the tarball and move the Velero binary to** `/usr/local/bin`:
        
        ```plaintext
        tar -xvf velero-v1.7.0-linux-amd64.tar.gz -C /tmp
        sudo mv /tmp/velero-v1.7.0-linux-amd64/velero /usr/local/bin
        ```
        
        This extracts the tarball and moves the Velero binary to a directory in your `PATH`.
        
    3. **Verify the installation**:
        
        ```plaintext
        velero version
        ```
        
* **Step 2: AWS Credentials Setup**
    
    Velero needs AWS credentials to interact with your S3 bucket or other AWS services.
    
    1. Create a file (e.g., `credentials-velero`) with your AWS credentials:
        
        ```plaintext
        [default]
        aws_access_key_id=YOUR_ACCESS_KEY_ID
        aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
        ```
        
        Make sure to replace `YOUR_ACCESS_KEY_ID` and `YOUR_SECRET_ACCESS_KEY` with your actual AWS credentials.
        
    2. You can get your AWS credentials from the **IAM Console** in the AWS Management Console. Ensure that the IAM user has sufficient permissions (like S3 and EC2).
        
* **Step 3: Velero Installation to Kubernetes Cluster**
    
    Now, you need to install Velero into your Kubernetes cluster and configure it to back up to your AWS S3 bucket.
    
    1. **Set Environment Variables** for your bucket and region:
        
        ```plaintext
        export BUCKET=k8-backup
        export REGION=ap-south-1
        ```
        
    2. **Install Velero on your Kubernetes Cluster**: Use the following command to install Velero, specifying your **AWS** provider, backup bucket, and credentials file:
        
        ```plaintext
        velero install \
            --provider aws \
            --plugins velero/velero-plugin-for-aws:v1.3.0 \
            --bucket $BUCKET \
            --backup-location-config region=$REGION \
            --snapshot-location-config region=$REGION \
            --secret-file ./credentials-velero
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232483438/566e459b-7be4-49cc-8729-7bf5ff0ba03b.png align="center")
        
        * `--provider aws`: Specifies that you are using AWS as your cloud provider.
            
        * `--plugins velero/velero-plugin-for-aws:v1.3.0`: The AWS plugin Velero will use.
            
        * `--bucket $BUCKET`: The name of your S3 bucket where the backups will be stored.
            
        * `--backup-location-config region=$REGION`: The AWS region where the S3 bucket is located.
            
        * `--snapshot-location-config region=$REGION`: The region where the snapshots will be taken.
            
        * `--secret-file ./credentials-velero`: Path to the credentials file you created earlier.
            
    3. **Verify Velero Installation**: After running the above command, check that the Velero pods are running properly in the `velero` namespace:
        
        ```plaintext
        kubectl get pods -n velero
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232498335/b4f95b12-2b25-499e-aef4-a8c7ed4375e1.png align="center")
        
* **Step 4: Creating a Kubernetes Application**
    
    For demonstration purposes, you’ll create a simple application that Velero can back up.
    
    1. **Create a simple NGINX deployment**:
        
        ```plaintext
        kubectl create deployment testing --image=nginx --replicas=2
        ```
        
    2. **Expose the deployment as a service**:
        
        ```plaintext
        kubectl expose deployment testing --name=test-srv --type=NodePort --port=80
        ```
        
    3. **Port-forward to access the application** (for local testing):
        
        ```plaintext
        kubectl port-forward svc/test-srv 8000:80
        ```
        
* **Step 5: Create a Backup of the Cluster**
    
    Now, you can back up your Kubernetes cluster using Velero.
    
    1. **Verify Backup Locations**:
        
        ```plaintext
        velero backup-location get
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232532782/51b47e70-1e69-4dac-8cbe-4a828444e796.png align="center")
        
        This will show the configured backup location (in this case, your S3 bucket).
        
    2. **Create a Backup**: Run the following command to create a backup of your cluster:
        
        ```plaintext
        velero backup create cluster-backup
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232563220/66bfbef8-dbc2-4ae9-a44e-e1ca1604f3a6.png align="center")
        
        Velero will now take a backup of the cluster, including Kubernetes resources (deployments, services, etc.) and persistent volumes.
        
        You can check the backup status using:
        
        ```plaintext
        velero backup get
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232579233/f04aabab-c727-4171-847a-b95b22560e8f.png align="center")
        
* **Step 6: Restore the Backup**
    
    If something goes wrong, or you want to restore the backup, you can do so with the following steps:
    
    1. **Restore from a Backup**: To restore from the backup you created earlier, run the following command:
        
        ```plaintext
        velero restore create new-backup-restore --from-backup cluster-backup
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232599104/65cb2028-3648-4241-bdd4-043e13709d9b.png align="center")
        
        This will initiate the restoration of all resources that were backed up in the `cluster-backup` backup.
        
    2. **Check the Restore Status**: You can monitor the restore process:
        
        ```plaintext
        velero restore get
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731232621356/03f1ccd3-4ff5-41ef-ac34-06b52a7308c2.png align="center")
        
* **Step 7: Verify the Restore**
    
    After the restore process completes, you can verify that the resources (like the `nginx` deployment and service) are successfully restored:
    
    1. **Check if the pods are restored**:
        
        ```plaintext
        kubectl get pods
        ```
        
    2. **Check if the service is restored**:
        
        ```plaintext
        kubectl get svc
        ```
        
    3. **Port-forward and verify the application is running**:
        
        ```plaintext
        kubectl port-forward svc/test-srv 8000:80
        ```
        
    
    Now you can access the application again on [`http://localhost:8000`](http://localhost:8000)
    
* **Advantages:**
    
    1. **Comprehensive Backup**: Backs up Kubernetes resources (pods, services, deployments) and persistent volumes.
        
    2. **Cloud Integration**: Supports cloud storage providers like **AWS S3**, **Azure Blob Storage**, and **Google Cloud Storage**.
        
    3. **Multi-Cluster Support**: Can back up and restore across multiple clusters or regions.
        
    4. **Automated Backups**: Supports scheduled, incremental, and automated backups.
        
    5. **Namespace-Level Backups**: Allows for selective backups and restores at the namespace level.
        
    6. **Persistent Volume Backup**: Can back up persistent volumes with **Restic** for cloud-managed clusters.
        
    7. **Disaster Recovery**: Enables disaster recovery with cross-region and cross-cloud restore capabilities.
        
    8. **Easy Setup and Use**: Simple CLI interface and easy-to-understand commands for users.
        
    
    **Disadvantages:**
    
    1. **Resource Overhead**: Velero introduces additional overhead in terms of running backup and restore processes in the cluster.
        
    2. **Limited to Kubernetes**: Does not back up non-Kubernetes resources (e.g., external databases, external storage).
        
    3. **Cloud Provider Dependency**: Relies on cloud storage, so backup costs and performance may depend on the cloud provider.
        
    4. **Initial Setup Complexity**: Setup and configuration, especially for multi-cluster or cloud integrations, can be complex.
        
    5. **Large Backup Sizes**: Can have large backup sizes, especially with persistent volumes, increasing storage and retrieval costs.
        
    6. **Backup Duration**: Large backups may take significant time to complete, affecting cluster performance during the process.
        

Conclusion:

In this blog, we've explored two essential backup solutions for Kubernetes: **etcdctl** for self-managed clusters and **Velero** for cloud-managed clusters.

**etcdctl** is ideal for backing up the **etcd** database in self-managed Kubernetes setups, but it's limited to control plane data and doesn’t handle workloads or persistent volumes.

On the other hand, **Velero** offers a more comprehensive solution, allowing you to back up not just the **etcd** data but also **Kubernetes resources**, **persistent volumes**, and **cloud-specific configurations**, making it the go-to choice for cloud-managed clusters like **EKS**, **AKS**, and **GKE**.

Both tools are critical for disaster recovery, and understanding how and when to use each is a key skill for Kubernetes professionals, especially for the **Certified Kubernetes Administrator (CKA)** exam. With the right backup strategies, you can ensure the availability and resilience of your Kubernetes environments.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
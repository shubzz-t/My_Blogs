---
title: "Highly Available Kubernetes Cluster Setup"
datePublished: Sun Sep 08 2024 11:07:10 GMT+0000 (Coordinated Universal Time)
cuid: cm0tgyvv800040ajta89uh4nz
slug: highly-available-kubernetes-cluster-setup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725360491065/0d5db8b2-7f72-4ac2-aaf7-e009d60d13b0.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1725793606635/419e6734-88c9-4477-8f4e-9b97a9ef5c70.png
tags: kubernetes, devops, devops-articles, high-availability

---

In today’s digital world, ensuring that your applications are always available is crucial. A highly available Kubernetes cluster helps you achieve this by maintaining operations despite failures, scaling efficiently, and recovering quickly from disasters. This blog will guide you through the essentials of creating a highly available Kubernetes cluster, covering best practices and strategies to keep your services robust and reliable. Whether you’re setting up a new cluster or enhancing an existing one, this guide will help you build a resilient infrastructure that meets your needs.

**Why we need highly available kubernetes cluster?**

1. **Minimize Downtime:** Ensures services remain operational even if components fail.
    
2. **Scale Effectively:** Handles increased load and adapts to changes seamlessly.
    
3. **Recover from Disasters:** Provides resilience against regional outages and failures.
    
4. **Enhance Reliability:** Offers fault tolerance and a better user experience.
    
5. **Meet Compliance and SLAs:** Helps in meeting strict up time requirements and regulatory standards.
    

Reference Repo: [https://github.com/jaiswaladi246/HA-K8-Cluster.git](https://github.com/jaiswaladi246/HA-K8-Cluster.git)

Now we will start with creating the highly available cluster:

**Prerequisites:**

1. AWS Account
    
2. 3 master node (Ubuntu - t2.medium - 25 GB Storage)
    
3. 2 worker node (Ubuntu - t2.medium- 25 GB Storage)
    
4. 1 Load Balancer Node (Ubuntu - t2.medium- 25 GB Storage)
    

### Step 1: Prepare Load Balancer Node:

* SSH into the Load Balancer Node.
    
* Run update command
    
    ```bash
    sudo apt update
    ```
    
* Install HA proxy:
    
    ```bash
    sudo apt-get install -y haproxy
    ```
    
* Configure HA proxy by editing the HA proxy configuration file located at (`/etc/haproxy/haproxy.cfg`):
    
    ```bash
    sudo nano /etc/haproxy/haproxy.cfg
    ```
    
* Add the below configuration to the above opened file:
    
    ```plaintext
    frontend kubernetes-frontend
        bind *:6443
        option tcplog
        mode tcp
        default_backend kubernetes-backend
    
    backend kubernetes-backend
        mode tcp
        balance roundrobin
        option tcp-check
        server master1 <MASTER1_IP>:6443 check
        server master2 <MASTER2_IP>:6443 check
        server master3 <MASTER2_IP>:6443 check
    ```
    
* Restart HA proxy:
    
    ```bash
    sudo systemctl restart haproxy
    ```
    

### Step 2: Preparing Master and Worker Nodes:

* Run the below script on all master and worker nodes. This will install docker, kubeadm, kubelet, kubectl.
    
    ```bash
    sudo apt-get update
    sudo apt install docker.io -y
    sudo chmod 666 /var/run/docker.sock
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
    sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    sudo apt install -y kubeadm=1.30.0-1.1 kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
    ```
    

### Step 3: Initialize the First Master Node:

* SSH into the first master node and run the below command:
    
    ```bash
    #Replace the load balancer ip with your ip 
    sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_IP:6443" --upload-certs --pod-network-cidr=10.244.0.0/16
    ```
    
* After running above command you will get the command to join the master nodes as well as to join worker nodes which will be used in **@Step4 and @Step 5**.
    
* **Set up kubeconfig for the first master node:**
    
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    
* **Install Calico network plugin:**
    
    ```bash
    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    ```
    
* **Install Ingress-NGINX Controller:**
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
    ```
    

### Step 4: Join Second Third Master Node:

* Get the join command and certificate key from the first master node. Which we got in above step.
    
* Run the join command on the second master node and third master node so that they can join the cluster.
    
* Set up kubeconfig for the second master node.
    
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    

### Step 5: Joining the worker nodes:

* Get the join command from first master node which we got in **@Step 3.**
    
* Run the join command on each worker node that we want in our cluster.
    

Step 6: Verify:

* Check the status of all nodes as to ensure all nodes are available using:
    
    ```bash
    kubectl get nodes
    ```
    
* Check status of all pods:
    
    ```bash
    kubectl get pods --all-namespaces
    ```
    

Here we are done with setting up our cluster. Next we will be verifying our cluster whether it is working as expected which is optional step.

### Step 6: Cluster Verification:

1. Verify Etcd Cluster Health:
    
    * Install ectdctl using apt on all master nodes:
        
        ```bash
        sudo apt-get update
        sudo apt-get install -y etcd-client
        ```
        
    * Verify health of cluster by running below command on all master nodes:
        
        ```bash
        ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key endpoint health
        ```
        
    * After above command you will get the status as healthy that means your cluster is in good state.
        
2. Verify HAProxy Functionality:
    
    * Configure the HAProxy for stats add below code to file `/etc/haproxy/haproxy.cfg`:
        
        ```bash
        listen stats
            bind *:8404
            mode http
            stats enable
            stats uri /
            stats refresh 10s
            stats admin if LOCALHOST
        ```
        
    * Restart the HAProxy to use configuration using:
        
        ```bash
        sudo systemctl restart haproxy
        ```
        
    * Check HAProxy Stats at below url:
        
        `http://<LOAD_BALANCER_IP>:8404`
        
3. Test High Availability of Cluster:
    
    * Firstly fail master node by stopping the kubelet service and Docker containers on one of the master nodes to simulate a failure using command:
        
        ```bash
        sudo systemctl stop kubelet
        sudo docker stop $(sudo docker ps -q)
        ```
        
    * Verify the cluster functionality by running command from the remaining master node:
        
        ```bash
        kubectl get nodes
        kubectl get pods --all-namespaces
        ```
        
    * The cluster should still show the remaining nodes as Ready, and the Kubernetes API should be accessible.
        
    * Ensure that HAProxy is routing traffic to the remaining master node. Check the stats page or use curl to test.
        
        ```bash
        curl -k https://<LOAD_BALANCER_IP>:6443/version
        ```
        

### Conclusion:

Building a highly available Kubernetes cluster is crucial for maintaining service reliability and resilience. By focusing on redundancy, load balancing, automated failover, and robust monitoring, you can ensure your applications stay operational and scalable. With ongoing maintenance and testing, you'll be well-prepared to handle disruptions and meet both performance and compliance needs.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

**Streamline, Deploy, Succeed-- Devops Made Simple!☺️**
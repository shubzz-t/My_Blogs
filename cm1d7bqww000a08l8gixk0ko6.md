---
title: "Comprehensive DevOps Monitoring: Prometheus, Node Exporter, Blackbox & Alert manager with Email Notifications"
datePublished: Sun Sep 22 2024 06:32:38 GMT+0000 (Coordinated Universal Time)
cuid: cm1d7bqww000a08l8gixk0ko6
slug: comprehensive-devops-monitoring-prometheus-node-exporter-blackbox-alert-manager-with-email-notifications
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1726948619784/6e1c6834-1752-46dd-bd7d-178158810bad.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1726986746403/fb736000-aa42-4e96-b9bd-b7697ab8a220.png
tags: cloud, monitoring, devops, prometheus, alertmanager, blackbox-exporter, nodeexporter

---

In this project, we will build a comprehensive DevOps monitoring solution using **Blackbox Exporter**, **Node Exporter**, **Alert manager**, and **Prometheus**. The goal is to set up a robust monitoring system that ensures high availability and performance for applications by continuously tracking system metrics and availability.

We will configure the **Blackbox Exporter** to monitor endpoints and ensure service uptime, while the **Node Exporter** will be used to collect key metrics from system hardware. **Prometheus** will act as the core monitoring and alerting tool, scraping the metrics and storing them efficiently. Additionally, we will configure **Alert manager** to handle alerts based on predefined conditions, including setting up **email notifications** for prompt issue resolution.

This project demonstrates the full monitoring lifecycle for a system, making it an essential part of any DevOps workflow, ensuring stability, performance, and rapid response to system failures.

**Prerequisites:**

* 2 - t2.medium ec2 with 20 GB each. (say instance 1 as monitoring and 2 as app vm)
    
* Deploy one web page using apache or nginx on one instance.
    

**Information:**

* Prometheus port: 9090
    
* BlackBox Exporter port: 9115
    
* Node Exporter port: 9100
    
* Alert Manager port: 9093
    

### 1\. Installing and Setup Monitoring tools:

* SSH into the monitoring ec2 instance and update the packages using:
    
    ```bash
    sudo apt update
    ```
    
* **Download prometheus:**
    
    1. Go to prometheus.io
        
    2. Copy prometheus download link and download using:
        
        ```bash
        #Download the prometheus 
        wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
        
        #Untar the prometheus downloaded package
        tar -xvf prometheus.xyz.tar.gz
        
        #Rename unzip file to prometheus
        mv prometheus.xyz.tar.gz/ prometheus
        ```
        
* **Download BlackBox Exporter:**
    
    1. Go to prometheus.io.
        
    2. Copy BlackBox exporter download link and download using:
        
        ```bash
        #Download blackbox exporter using the link
        wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
        
        #Untar the tar file
        tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
        
        #Rename unzip file to blackbox for ease
        mv blackbox_exporter-0.25.0.linux-amd64.tar.gz blackbox
        ```
        
* **Download Alert Manager:**
    
    1. Go to prometheus.io.
        
    2. Copy Alert exporter download link and download using:
        
        ```bash
        #Download Alert Manager using the link
        wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
        
        #Untar the tar file
        tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
        
        #Rename unzip file to blackbox for ease
        mv alertmanager-0.27.0.linux-amd64.tar.gz alertmanager
        ```
        
    
    **Setup Alert rules in Prometheus:**
    
    1. Go into the prometheus folder:
        
        ```bash
        cd prometheus
        ```
        
    2. Create file `alert_rules.yml` and put the below content in it.
        
        ```bash
        ---
        groups:
          - name: alert_rules
            rules:
              - alert: InstanceDown
                expr: up == 0
                for: 1m
                labels:
                  severity: critical
                annotations:
                  summary: 'Endpoint {{ $labels.instance }} down'
                  description: >-
                    {{ $labels.instance }} of job {{ $labels.job }} has been down for
                    more than 1 minute.
              - alert: WebsiteDown
                expr: probe_success == 0
                for: 1m
                labels:
                  severity: critical
                annotations:
                  description: 'The website at {{ $labels.instance }} is down.'
                  summary: Website down
              - alert: HostOutOfMemory
                expr: node_memory_MemAvailable / node_memory_MemTotal * 100 < 25
                for: 5m
                labels:
                  severity: warning
                annotations:
                  summary: 'Host out of memory (instance {{ $labels.instance }})'
                  description: |-
                    Node memory is filling up (< 25% left)
                     VALUE = {{ $value }}
                     LABELS: {{ $labels }}
              - alert: HostOutOfDiskSpace
                expr: >-
                  (node_filesystem_avail{mountpoint="/"} * 100) /
                  node_filesystem_size{mountpoint="/"} < 50
                for: 1s
                labels:
                  severity: warning
                annotations:
                  summary: 'Host out of disk space (instance {{ $labels.instance }})'
                  description: |-
                    Disk is almost full (< 50% left)
                     VALUE = {{ $value }}
                     LABELS: {{ $labels }}
              - alert: HostHighCpuLoad
                expr: >-
                  (sum by
                  (instance)(irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m])))
                  > 80
                for: 5m
                labels:
                  severity: warning
                annotations:
                  summary: 'Host high CPU load (instance {{ $labels.instance }})'
                  description: |-
                    CPU load is > 80%
                     VALUE = {{ $value }}
                     LABELS:{{ $labels }}
              - alert: ServiceUnavailable
                expr: 'up{job="node_exporter"} == 0'
                for: 2m
                labels:
                  severity: critical
                annotations:
                  summary: 'Service Unavailable (instance {{ $labels.instance }})'
                  description: |-
                    The service {{ $labels.job }} is not available
                     VALUE = {{ $value }}
                     LABELS: {{ $labels }}
              - alert: HighMemoryUsage
                expr: (node_memory_Active / node_memory_MemTotal) * 100 > 90
                for: 10m
                labels:
                  severity: critical
                annotations:
                  summary: 'High Memory Usage (instance {{ $labels.instance }})'
                  description: |-
                    Memory usage is > 90%
                     VALUE = {{ $value }}
                     LABELS: {{ $labels }}
              - alert: FileSystemFull
                expr: (node_filesystem_avail / node_filesystem_size) * 100 < 10
                for: 5m
                labels:
                  severity: critical
                annotations:
                  summary: 'File System Almost Full (instance {{ $labels.instance}})'
                  description: |-
                    File system has < 10% free space
                     VALUE = {{ $value }}
                     LABELS: {{ $labels }}
        ```
        
    3. Now we will edit the `prometheus.yml`.
        
        ```bash
        vi prometheus.yml
        ```
        
    4. Give our rule file name under the rule\_files section as alert\_rules.yml
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726980949265/3641f56b-66fe-429c-afaa-8a925d743475.png align="center")
        
    5. Start/Restart Prometheus so that alert rules will reflect. It will run on port &lt;ec2-publicip-prometheus&gt;:9090.
        
        ```bash
        #To stop previously running prometheus service 
        prgrep prometheus
        
        #Use the above command output as id
        kill id
        
        #Start the prometheus 
        ./prometheus &
        ```
        

### 2\. Install Node Exporter on App VM:

* You need to install the node exporter to capture and send the metrics of the running web application to prometheus, for that node exporter should be installed on app vm where your web application is running.
    
* **Download Node Exporter:**
    
    1. Go to prometheus.io.
        
    2. Copy Node Exporter download link and download using:
        
        ```bash
        #Download Alert Manager using the link
        wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
        
        #Untar the tar file
        tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
        
        #Rename unzip file to blackbox for ease
        mv node_exporter-1.8.2.linux-amd64.tar.gz node_exporter
        
        #Change directory to prometheus
        cd node_exporter
        
        #Run prometheus in background
        ./node_exporter &
        ```
        

### 3\. Configure Alert Manager Node and Blackbox Exporter in Prometheus:

* **Go into the prometheus folder and open prometheus.yml:**
    
    ```bash
    cd prometheus
    
    #Open prometheus.yml
    vi prometheus.yml
    ```
    
* **Configure Alert Manager:**
    
    1. Under alert manager configuration in target section provide &lt;public-ip-alert-manager&gt;:9093 where alert manager is running.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726981389222/a5ff9512-3043-48e5-90d1-bb1af942a96f.png align="center")
        
* **Configure Node Exporter:**
    
    1. Under scrape \_configs section we need to add the job for Node exporter.
        
        ```bash
        - job_name: node_exporter
          static_configs:
              - targets: ['<public-ip-nodeexporter>:9100']
        ```
        
    2. Restart the prometheus if already started and you will see that the node exporter is added to prometheus targets section.
        
* **Configure BlackBox exporter:**
    
    1. Under scrape \_configs section we need to add the job for BlackBox exporter.
        
        ```bash
          - job_name: blackbox
            metrics_path: /probe
            params:
              module:
                - http_2xx
            static_configs:
              - targets:
                  - http://prometheus.io
                  - https://prometheus.io
                  - http://your-website-ip:8080
            relabel_configs:
              - source_labels:
                  - __address__
                target_label: __param_target
              - source_labels:
                  - __param_target
                target_label: instance
              - target_label: __address__
                replacement: <public-ip-blackbox>:9115
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726981838004/346a80fb-37bd-49ee-8f80-f8ae985d11be.png align="center")
        
    2. Restart/Start the prometheus and blackbox exporter, you will see that the blackbox exporter is added to prometheus targets section.
        

### 4\. Configure alerts using Alert Manager:

* **Go inside folder alert manager:**
    
    ```bash
    cd alert_manager
    
    #Remove the previous alertmanager.yml
    rm alertmanager.yml
    
    #Create new alertmanager.yml
    vi alertmanager.yml
    ```
    
* **Paste the below content in** `alertmanager.yml`
    
    ```yaml
    ---
    route:
      group_by: [ 'alertname' ]
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 1h
      receiver: 'email-notifications'
    receivers:
      - name: 'email-notifications'
        email_configs:
          - to: shubhamtaware2001@gmail.com
            from: monitoring@gmail.com
            smarthost: 'smtp.gmail.com:587'
            auth_username: shubham.ajspire@gmail.com
            auth_identity: shubham.ajspire@gmail.com
            auth_password: "wsdl imvc auzb kihw"
            send_resolved: true
    
    inhibit_rules:
      - source_match:
          severity: 'critical'
        target_match:
          severity: 'warning'
        equal: ['alertname', 'dev', 'instance']
    ```
    
* **Start the alert manager using:**
    
    ```bash
    cd alertmanager
    
    #Starting the alert manager in background
    ./alertmanager &
    ```
    

Here we are done with the configuration part now we will test, weather the tools configured are working.

### 5\. Testing the Alert Manager:

* **By stopping the web application:**
    
    Stop the running web application on the VM.Go to prometheus &gt; Alert &gt; Website Down alert will be fired and you will get the mail to the configured mail id.
    
* **By taking down node exporter:**
    
    We will stop the node exporter service so that it will fire the email for Service unavailable and instance down.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726985077730/72ade974-9bc2-42de-9e25-bd2204f75f00.png align="center")
    
    * Below is the screenshot showcasing the alert that we got through email regarding the service unavailable.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726985146267/27db979f-3813-407e-bc95-b36c95e6bda6.png align="center")
        

### Conclusion:

In this project, we successfully set up a full-fledged DevOps monitoring solution using **Prometheus**, **Blackbox Exporter**, **Node Exporter**, and **Alertmanager**. This comprehensive setup enables real-time system and application monitoring, allowing you to track vital metrics and ensure service uptime. By integrating **Alertmanager** with **email notifications**, we’ve established a proactive alerting system that ensures timely responses to potential issues, helping teams maintain high system reliability and performance.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
---
title: "Efficient System Monitoring Using Shell Scripts: A Real-Time DevOps Solution"
datePublished: Sat Oct 26 2024 09:48:07 GMT+0000 (Coordinated Universal Time)
cuid: cm2pza41e000109jledkr67h1
slug: efficient-system-monitoring-using-shell-scripts-a-real-time-devops-solution
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1728932732414/0225f179-7538-43e2-88d4-e0454edb4611.gif
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1729936071712/9adbb576-6556-4409-b6e1-fabfdc028407.gif
tags: linux, automation, devops, shell-script

---

In the fast-paced world of DevOps, real-time system monitoring and maintenance are crucial to ensure continuous delivery and optimal system performance. This blog explores how shell scripts can be utilized by DevOps engineers to automate essential system checks such as disk usage, service health, network connectivity, database backup, and more. By leveraging these scripts, you can simplify routine tasks, reduce manual intervention, and proactively address system issues before they become critical.

Github Repository: [https://github.com/shubzz-t/Scripts\_Devops.git](https://github.com/shubzz-t/Scripts_Devops.git)

### 1\. Backup Script:

* This script automates the process of creating backups for critical files or directories on your system
    
    ```bash
    #!/bin/bash
    
    SOURCE="/home/ubuntu/mydata"
    DESTINATION="/home/ubuntu/backup/"
    DATE=$(date +%Y-%m-%d_%H-%M-%S)
    
    mkdir -p $DESTINATION/$DATE
    cp -r $SOURCE $DESTINATION/$DATE
    echo "BACKUP COMPLETED ON $DATE"
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728908223609/82fe84fd-9d5c-41ea-b71d-226747cb0612.png align="center")
    
* Explanation:
    
    1. `SOURCE="/home/ubuntu/mydata"`: Defines the directory to back up.
        
    2. `DESTINATION="/home/ubuntu/backup/"`: Defines where the backup will be stored.
        
    3. `DATE=$(date +%Y-%m-%d_%H-%M-%S)`: Gets the current date and time for the backup folder name.
        
    4. `mkdir -p $DESTINATION/$DATE`: Creates the backup folder with a timestamp.
        
    5. `cp -r $SOURCE $DESTINATION/$DATE`: Recursively copies the source directory to the backup folder.
        
    6. `echo "BACKUP COMPLETED ON $DATE"`: Displays the completion message with the backup time.
        

### 2\. Disk Usage Script

* This script is important for monitoring disk usage on servers. By setting a threshold, it helps prevent potential issues caused by high disk utilization, such as system slowdowns or crashes.
    
    ```bash
    #!/bin/bash
    
    THRESHOLD=80
    
    df -H | grep -vE '^Filesystem|cdrom|tmpfs' | awk '{ print $5 " " $1}' |
    	while read output;
    	do
    		usage=$(echo $output | awk '{ print $1 }' | cut -d'%' -f1) 
    		partition=$(echo $output | awk '{print $2}')
    		if [ $usage -ge $THRESHOLD ]; then
    			echo "Warning!!! Disk usage on $partition is at ${usage}%"
    		fi
    	done
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728910696956/7a65fa8e-d475-4ee7-a5d4-a7280cd89806.png align="center")
    
* Explanation:
    
    1. `THRESHOLD=80`: Sets the threshold for disk usage to 80%.
        
    2. `df -H`: Displays disk space usage in a human-readable format.
        
    3. `grep -vE '^Filesystem|cdrom|tmpfs'`: Filters out lines that start with 'Filesystem', and excludes CD-ROM and tmpfs mounts.
        
    4. `awk '{ print $5 " " $1}'`: Extracts the fifth column (usage percentage) and the first column (partition name) from the filtered output.
        
    5. `while read output; do ... done`: Loops through each line of output.
        
    6. `usage=$(echo $output | awk '{ print $1 }' | cut -d'%' -f1)`: Retrieves the usage percentage without the '%' symbol.
        
    7. `partition=$(echo $output | awk '{print $2}')`: Extracts the partition name.
        
    8. `if [ $usage -ge $THRESHOLD ]; then ... fi`: Checks if the usage percentage is greater than or equal to the defined threshold.
        
    9. `echo "Warning!!! Disk usage on $partition is at ${usage}%"`: Displays a warning message if the usage exceeds the threshold.
        

### 3\. Service Health Check Script

* The service health check script ensures that critical services, like Nginx, remain operational by automatically checking their status and restarting them if they are down. This proactive monitoring minimizes downtime and enhances overall system reliability.
    
    ```bash
    #!/bin/bash
    
    SERVICE="nginx"
    
    if systemctl is-active --quiet $SERVICE; then
            echo "$SERVICE is running"
    else
            echo "$SERVICE is not running"
            sudo systemctl start $SERVICE
    fi
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728920517158/62f7c173-1b51-4fd9-8b4e-fefbef62a385.png align="center")
    
* Explanation:
    
    1. `SERVICE="nginx"`: Sets the variable `SERVICE` to the name of the service being monitored (Nginx).
        
    2. `if systemctl is-active --quiet $SERVICE; then`: Checks if the Nginx service is currently active without producing output; if true, proceeds with the following commands.
        
    3. `echo "$SERVICE is running"`: Prints a message indicating that the Nginx service is running.
        
    4. `else`: Defines an alternative action if the service is not running.
        
    5. `echo "$SERVICE is not running"`: Prints a message indicating that the Nginx service is not running.
        
    6. `systemctl start $SERVICE`: Starts the Nginx service if it was found to be inactive.
        

### 4\. Network Connectivity Script

* This script checks the network connectivity to a specified host (in this case, Google) and logs the result to an output file. It helps in monitoring the status of network connections, ensuring services are reachable and operational, which is essential for troubleshooting network issues.
    
    ```bash
    #!/bin/bash
    
    HOST="www.google.com"
    OUTPUT_FILE="/home/ubuntu/output.txt"
    
    if ping -c 1 $HOST &> /dev/null
    then
            echo "$HOST is reachable" >> $OUTPUT_FILE
    else
            echo "$HOST is not reachable" >> $OUTPUT_FILE
    fi
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728921857347/44300c31-0f72-4d59-99f9-c95f0b20de22.png align="center")
    
* Explanation:
    
    1. `HOST="`[`google.com`](http://google.com)`"`: Sets the variable `HOST` to the target domain (Google) to be pinged for connectivity checks.
        
    2. `OUTPUT_FILE="/home/ubuntu/output.txt"`: Defines the file path where the results of the connectivity check will be stored.
        
    3. `if ping -c 1 $HOST &> /dev/null`: Pings the specified host once (`-c 1`), redirecting both stdout and stderr to `/dev/null` to suppress output; if successful, the script proceeds.
        
    4. `then`: Begins the block of commands to execute if the ping command is successful (host is reachable).
        
    5. `echo "$HOST is reachable" >> $OUTPUT_FILE`: Appends a message indicating that the host is reachable to the output file.
        
    6. `else`: Defines the alternative actions to take if the host is not reachable.
        
    7. `echo "$HOST is not reachable" >> $OUTPUT_FILE`: Appends a message indicating that the host is not reachable to the output file.
        

### 5\. Database Backup Script:

* This script automates the process of backing up a MySQL database, ensuring data integrity and safety by regularly saving the database state to a file. It helps in disaster recovery and data restoration in case of data loss or corruption.
    
    ```bash
    #!/bin/bash
    
    DB_NAME="mydb"
    BACKUP_DIR="/home/ubuntu/backup/"
    DATE=$(date +%Y-%m-%d_%H-%M-%S)
    
    mysqldump -u root -p $DB_NAME > $BACKUP_DIR/$DB_NAME-$DATE.sql
    echo "DATABASE BACKUP DONE TO: $BACKUP_DIR/$DB_NAME-$DATE.sql"
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728923038824/c6a9b5e3-4138-44a1-8123-6b7b1eb4d847.png align="center")
    
* Explanation:
    
    1. `DB_NAME="mydatabase"`: Sets the variable `DB_NAME` to the name of the database that will be backed up.
        
    2. `BACKUP_DIR="/path/to/backup"`: Defines the path to the directory where the backup files will be stored.
        
    3. `DATE=$(date +%Y-%m-%d_%H-%M-%S)`: Assigns the current date and time in the specified format to the variable `DATE`, which will be used in the backup file name.
        
    4. `mysqldump -u root -p $DB_NAME > $BACKUP_DIR/$DB_NAME-$DATE.sql`: Executes the `mysqldump` command to create a backup of the specified database (`$DB_NAME`) using the root user, prompting for the password, and saves the output to a file in the backup directory with a timestamped name.
        
    5. `echo "Database backup completed: $BACKUP_DIR/$DB_NAME-$DATE.sql"`: Outputs a message indicating that the database backup has been completed successfully and specifies the location of the backup file.
        

### 6\. System Uptime Script

* This script provides a quick way to monitor the system's uptime, helping administrators assess system stability and performance over time, and identify potential issues related to unexpected reboots.
    
    ```bash
    #!/bin/bash
    
    uptime -p
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728923396467/c2e577dc-4c81-4244-8f07-a3a6f1b0b868.png align="center")
    
* Explanation:
    
    1. `uptime -p`: Displays the system uptime in a human-readable format, showing how long the system has been running since the last boot.
        

### 7\. Listening ports script:

* This script helps administrators monitor active network services by identifying which ports are currently listening, ensuring that only necessary services are open, which is critical for system security and performance
    
    ```bash
    #!/bin/bash
    
    netstat -tuln | grep LISTEN
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728924912294/a657f088-af0f-47ae-9814-35182e1fecde.png align="center")
    
* Explanation:
    
    1. `netstat -tuln`: Displays all active TCP and UDP listening ports without resolving service names, showing the port numbers instead.
        
    2. `grep LISTEN`: Filters the output to only show ports that are in a "LISTEN" state, indicating they are actively waiting for connections.
        

### 8\. Update Clean Packages:

* This script ensures that the system remains up-to-date with the latest security patches, performance improvements, and cleans up unnecessary packages to free up space and maintain system efficiency.
    
    ```bash
    #!/bin/bash
    
    apt-get update && apt-get upgrade -y && apt-get autoremove -y && apt-get clean
    echo "SYSTEM PACKAGES UPDATED AND CLEANED UP!!!"
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728926085463/7df4c83c-2eed-4ec4-95f5-45d81f581b13.png align="center")
    
* Explanation:
    
    1. `apt-get update`: Refreshes the package index to get the latest package versions.
        
    2. `apt-get upgrade -y`: Installs available updates for all packages without prompting.
        
    3. `apt-get autoremove -y`: Removes unnecessary packages that were installed as dependencies but are no longer needed.
        
    4. `apt-get clean`: Clears out the local repository of downloaded package files to free up space.
        
    5. `echo "System packages updated and cleaned up"`: Prints a confirmation message after completion.
        

### 9\. Http Response Time

* This script helps monitor the performance of websites by measuring the response times for multiple URLs, ensuring quick identification of potential slowdowns or outages.
    
    ```bash
    #!/bin/bash
    
    URLS=("https://www.google.com", "https://www.github.com")
    
    for URL in "${URLS[@]}"; do
            RESPONSE_TIME=$(curl -o /dev/null -s -w '%{time_total}\n' $URL)
            echo "RESPONSE TIME FOR $URL: $RESPONSE_TIME seconds"
    done
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728927141829/ba082995-e22c-4292-ba42-18a0c4008280.png align="center")
    
* Explanation:
    
    1. `URLS=("`[`https://www.google.com/`](https://www.devopsshack.com/)`" "`[`https://www.github.com/`](https://www.linkedin.com/)`")`: Defines an array of URLs to check.
        
    2. `for URL in "${URLS[@]}"`: Loops through each URL in the array.
        
    3. `curl -o /dev/null -s -w '%{time_total}\n' $URL`: Uses `curl` to silently measure the total time taken for the HTTP request and stores it in `RESPONSE_TIME`.
        
    4. `echo "Response time for $URL: $RESPONSE_TIME seconds"`: Prints the response time for each URL.
        

### 10\. Monitor System Process and Memory Usage:

* This script provides a snapshot of the top 10 processes consuming the most memory, helping identify resource-hogging processes and improve system performance.
    
    ```bash
    #!/bin/bash
    
    ps aux --sort=-%mem | head -n 10
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728928244507/e3f584c3-f4f8-4318-990c-5121753ca8bb.png align="center")
    
* Explanation:
    
    1. `ps aux --sort=-%mem`: Lists all running processes and sorts them by memory usage in descending order.
        
    2. `head -n 10`: Displays the top 10 processes from the sorted list.
        

### Conclusion

In this blog, we explored essential shell scripts for monitoring and managing various aspects of system health and performance, including backups, disk usage, service health, network connectivity, database backups, uptime, listening ports, package updates, HTTP response times, and system processes. These scripts empower DevOps engineers to automate routine tasks, ensure system stability, and quickly respond to potential issues. By integrating these checks into your infrastructure, you can enhance system efficiency, minimize downtime, and streamline maintenance efforts in real-time environments.

For more insightful content on technology, AWS, and DevOps, make sure to follow me for the latest updates and tips. If you have any questions or need further assistance, feel free to reach out—I’m here to help!

Streamline, Deploy, Succeed-- Devops Made Simple!☺️
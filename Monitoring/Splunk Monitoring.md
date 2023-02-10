# Splunk Monitoring

### **Downloading Splunk enterprise on your browser**

- Create an Account in [Splunk](https://www.splunk.com/?locale=en_us) website
- Click the **free Splunk** in the upper right corner
- Choose **Splunk Enterprise**
- Download the right one for your environment

### **Installing Splunk on your instance**

- Login as root
    
    ```bash
    sudo su -l root
    ```
    
- If you haven’t downloaded the Splunk application yet do this step
    
    ```bash
    wget -O splunk-9.0.2-17e00c557dc1-linux-2.6-x86_64.rpm "<https://download.splunk.com/products/splunk/releases/9.0.2/linux/splunk-9.0.2-17e00c557dc1-linux-2.6-x86_64.rpm>"
    ```
    
- Install the application
    
    ```bash
    # Locate the downloaded app
    rpm -i splunk-9.0.2-17e00c557dc1-linux-2.6-x86_64.rpm
    ```
    
- Add the Splunk bin file path into $PATH
    - Open your .bash_profile file in ~/.bash_profile(located in Root home Directory)
    
    ```bash
    cd /root
    ```
    
    - Append **:/opt/splunk/bin** in variable PATH so that we don’t have to specify an absolute path when running a Splunk command.
    
    ```bash
    vi .bash_profile
    
    # This will reload the file to apply the changes
    source .bash_profile
    ```
    
    ![Untitled.png](/Monitoring/Screenshots/Untitled.png)
    
- Start Splunk Enterprise and enable it
    
    ```bash
    # Start Splunk Enterprise and create administrator credentials
    splunk start --accept-license --answer-yes
    
    # Configure it to start at boot time
    splunk enable boot-start
    ```
    
- Create a new index for your server
    
    💡 The index is the repository for Splunk Enterprise data. Splunk Enterprise transforms incoming data into events, which it stores in indexes. For more detail, the index is like setting up a tag to separate the various fields in Splunk and used as a reference for our instance or server logs, or metrics.
    
    ```bash
    # Create Splunk index "<index-name>" for k8s logs
    
    splunk add index <index-name>
    
    #Sample: splunk add index server-field
    ```
    
- Create a new token for your server
    
    💡 The Splunk Web Framework uses tokens to provide a data-binding mechanism so that search managers and views can share a data value and stay in sync. For more details, tokens are used as a kind of credential by instances so that instances can connect to Splunk
    
    ```bash
    # Create a new HEC token
    splunk http-event-collector create <token-name> -uri <https://127.0.0.1:8089> -description "<Description>" -index <index-name>
    
    #Sample: splunk http-event-collector create web-token -uri <https://127.0.0.1:8089> -description "This is my first web-token" -index nginx-field
    
    # enable HTTP Event Collector in the global configuration
    splunk http-event-collector enable -uri <https://127.0.0.1:8089> -enable-ssl 0
    ```
    
    > *Uri is the localhost and port of Splunk logs. For more references to 127.0.0.1 follow this link. The default URI port number to use must be the management port(8089) of your Splunk server.*
    > 
    
    > *To have HEC listen and communicate over HTTPS rather than HTTP, SSL is enabled.*
    > 
- Update the **fields.conf** file
    
    ```
    # Update fields.conf on Splunk to make metadata fields searchable
    cat << EOF | sudo tee -a /opt/splunk/etc/system/local/fields.conf
    [namespace]
    INDEXED = true
    [pod]
    INDEXED = true
    [container_name]
    INDEXED = true
    [container_id]
    INDEXED = true
    [cluster_name]
    INDEXED = true
    EOF
    ```
    
- Restart Splunk to apply changes
    
    ```
    splunk restart
    ```
    
- Accessing Splunk UI in your browser
    
    > By default, Splunk will run on port 8000 for the web services and port 8089 for splunkd services.
    > 
    - Since we are using port 8000. You need to open it in the security group.
        
        ![Untitled 1.png](/Monitoring/Screenshots/Untitled%201.png)
        
    - Go to: **http://**:**8000**
        
        ![Untitled 2.png](/Monitoring/Screenshots/Untitled%202.png)
        
- Allowing indexes in the Splunk UI
    - Login using the credentials you created earlier
    - Go to **Settings** > **Data inputs**
        
        ![Untitled 3.png](/Monitoring/Screenshots/Untitled%203.png)
        
    - Click **HTTP Event Collector**
        
        💡 The HTTP Event Collector (HEC) enables you to send data over HTTP (or HTTPS) directly to Splunk Enterprise from your application. After you enable HEC, you can use HEC tokens in your app to send data to HEC.
        
        ![Untitled 4.png](/Monitoring/Screenshots/Untitled%204.png)
        
    - Edit and add **ALL** the indexes then **Save**
        
        ![Untitled 5.png](/Monitoring/Screenshots/Untitled%205.pngS)
        
- Adding Kubernetes cluster to Splunk
    - Install helm. You can refer to [this step](https://www.notion.so/Observability-and-Logging-Infrastructure-a248936838034716944fd69451726a36) on how to install helm
    - Add Splunk repo and create the values.yaml
        
        ```
        helm repo add splunk <https://splunk.github.io/splunk-connect-for-kubernetes/>
        helm show values splunk/splunk-connect-for-kubernetes > values.yaml
        
        ```
        
    - Add the required values to your values.yaml
        
        Index name is the index created previously, for this workload we called ours `server-field` . The host is the public IP DNS of the server where Splunk is installed. For the cluster name, we used `"minikube"`.
        
        ```
        global:
            logLevel: info
            splunk:
            hec:
        	      # Replace the values inside "<>"
        	      ### This part is truncated ###
        	      host: <FQDN of the instance where splunk is installed>
        	      #Port 8088 is the default port
        	      port: 8088
        	      token: <token generated from the Splunk UI>
        	      # get the token from spark UI > settings > data inputs > HTTP event collector
        	      protocol: http
        	      indexName: <indexName>
        	      ### This part is truncated ###
        	      kubernetes:
        	        clusterName: "<clusterName>"
        	      ## This part is truncated ###
        
        ```
        
    - Open port 8088 in your instance security groups to allow indexing of logs.
        
        Refer to [this step](https://www.notion.so/Observability-and-Logging-Infrastructure-a248936838034716944fd69451726a36) on how to open ports in the security group.
        
    - Apply the changes through helm
        
        ```
        helm install my-splunk-connect -f values.yaml splunk/splunk-connect-for-kubernetes
        
        # For succeeding changes in values.yaml run this instead:
        helm upgrade my-splunk-connect -f values.yaml splunk/splunk-connect-for-kubernetes
        
        ```
        
    - Create an index in your Splunk application. Run this command where your Splunk is installed
        
        > Now this index is for the logs of our application that we are watching, we called ours nginx-field
        > 
        
        ```
        splunk add index <indexName>
        splunk restart
        ```
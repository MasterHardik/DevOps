# Deploying a Website on AWS EC2 (Beginner's Guide)

This guide walks you through the process of setting up an EC2 instance on AWS, installing required software, and deploying a simple website using Apache HTTPD and Tomcat server. 

---

## Prerequisites
- AWS Free Tier account.
- Basic knowledge of SSH and Linux commands.
- SSH key pair to access the EC2 instance.

---

## Step 1: Launch an EC2 Instance
1. **Log in to AWS Management Console** and navigate to EC2.
2. Click on **Launch Instance**.
3. Configure the instance:
   - Select an Amazon Machine Image (AMI): **Amazon Linux 2**.
   - Instance type: **t2.micro** (Free Tier eligible).
   - Key pair: Select an existing one or create a new one.
   - Security Group:
     - Add rules for **SSH (port 22)**, **HTTP (port 80)**, and **Tomcat (port 8080)**.
4. Launch the instance and note the public IP or DNS.

---

## Step 2: Connect to Your Instance
1. Open a terminal and SSH into the instance:
   
   ```bash
   ssh -i your-key.pem ec2-user@<Public-IP>
   ```
2. Switch to the root user:
bash
   
   ```bash
   sudo su
   ```

## Step 3: Assign an *Elastic IP*
#### Why Assign an Elastic IP?
By default, AWS assigns a *dynamic public IP* to your `EC2` instance. This IP changes every time the instance is stopped and started. If you're hosting a website or an application, a dynamic IP means the address will keep changing, disrupting access to your application. Assigning an Elastic IP ensures your instance has a static, unchanging public IP, making it accessible even after restarts.

#### Steps to Assign an Elastic IP:
1. Go to the Elastic IPs section in the EC2 dashboard.
2. Click Allocate Elastic IP address and confirm the allocation.
3. Associate the Elastic IP with your EC2 instance:
   - Select the Elastic IP and click Actions > Associate Elastic IP address.
   - Choose your instance from the dropdown and save the changes.

Once done, use this Elastic IP to connect to your instance or access your website. This ensures consistent access, even if the instance is *restarted*.

## Step 4: Install Java (JDK)

1. Download and extract Amazon Corretto JDK:
   ```bash
   wget https://corretto.aws/downloads/resources/8.x/amazon-corretto-8-x64-linux-jdk.tar.gz
   tar -xvzf amazon-corretto-8-x64-linux-jdk.tar.gz
   sudo mv amazon-corretto-8.432.06.1-linux-x64 /usr/lib/jvm/
   ```

2. Set up environment variables:
   
   ```bash
   sudo vi /etc/profile
   ```

- Add the following lines:
   ```bash
   export JAVA_HOME=/usr/lib/jvm/amazon-corretto-8.432.06.1-linux-x64
   export PATH=$JAVA_HOME/bin:$PATH
   ```

3. Apply the changes:

   ```bash
   source /etc/profile
   ```

4. Verify the installation:

   ```bash
   java -version
   ```

## Step 5: Install and Configure Apache HTTPD

1. Install HTTPD:

   ```bash
   sudo yum install httpd -y
   ```
2. Start and enable the HTTPD service:

   ```bash
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```

3. Create a sample webpage:
   ```bash
   echo "<h1>Welcome to My Website</h1>" | sudo tee /var/www/html/index.html
   ```

## Step 6: Install and Configure Tomcat
1. Download and extract Tomcat:
   ```bash
   wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.81/bin/apache-tomcat-8.5.81.tar.gz
   sudo tar -xzvf apache-tomcat-8.5.81.tar.gz -C /opt
   sudo mv /opt/apache-tomcat-8.5.81 /opt/tomcat8
   ```

2. Set up environment variables for Tomcat:
   ```bash
   echo "export CATALINA_HOME=/opt/tomcat8" >> ~/.bashrc
   echo "export PATH=$CATALINA_HOME/bin:$PATH" >> ~/.bashrc
   source ~/.bashrc
   ```

3. Start Tomcat:
   ```bash
   sudo /opt/tomcat8/bin/startup.sh
   ```

4. Verify Tomcat: Navigate to `http://<Elastic-IP>:8080` in your browser. You should see the Tomcat welcome page.

## Step 7: Configure Security Groups

1. Navigate to the EC2 dashboard and edit the instance's security groups.

2. Add the following rules:
   - **Port 22 (SSH)**: Allow your IP.
   - **Port 80 (HTTP)**: Allow public access.
   - **Port 8080 (Tomcat)**: Allow public access.

3. Save and apply the rules.

## Step 8: Automate Startup and Shutdown

### Startup Commands:

   ```bash
   ssh -i aws-test-key.pem ec2-user@<Elastic-IP>
   sudo su
   sudo /opt/tomcat8/bin/startup.sh
   sudo systemctl start httpd
   ```

### Shutdown Commands:

   ```bash
   sudo systemctl stop httpd
   sudo /opt/tomcat8/bin/shutdown.sh
   exit
   ```

## Step 9: Verify the Setup

1. Open `http://<Elastic-IP>` in a browser to see the HTTPD webpage.

2. Access `http://<Elastic-IP>:8080` for Tomcat.

## Step 10: Troubleshooting: Access Issues with Tomcat Server

If you're having trouble accessing your Tomcat server (especially if it's running on port 8080), follow these steps to update the security group inbound rules:

1. **Go to the EC2 Dashboard**  
   - Navigate to the **EC2 dashboard** in your AWS Management Console.
   
2. **Select Your Instance**  
   - Click on **Instances** and choose the instance running your Tomcat server.

3. **Access Security Groups**  
   - In the **Description** section, find the **Security groups** field and click on the security group linked to your EC2 instance.

4. **Update Inbound Rules**  
   - In the security group details, click on the **Inbound rules** tab.
   - Click **Edit inbound rules**.

5. **Add the Following Rules:**

   - **Rule 1 (IPv4):**  
     - **Type:** Custom TCP  
     - **Port Range:** 8080  
     - **Source:** 0.0.0.0/0  
     - This rule allows HTTP access to your Tomcat server from any IPv4 address on port 8080.
   
   - **Rule 2 (IPv6):**  
     - **Type:** Custom TCP  
     - **Port Range:** 8080  
     - **Source:** ::/0  
     - This rule allows HTTP access to your Tomcat server from any IPv6 address on port 8080.

6. **Save Changes**  
   - After adding both rules, click **Save rules** to apply the changes.

These rules open port 8080 to the public, allowing you to access your Tomcat server through the browser using `http://<your-ec2-ip>:8080`.


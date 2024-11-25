# Adding Security Groups
### Resolving Access Issues with Tomcat Server

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

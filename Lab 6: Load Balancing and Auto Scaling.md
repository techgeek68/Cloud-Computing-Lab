# Implementing Load Balancing and Auto Scaling on AWS

---

**Objectives**

- Configure an Application Load Balancer to distribute incoming traffic.
- Set up an Auto Scaling group using a launch template.
- Implement scaling policies that respond to CloudWatch metrics.
- Test horizontal scaling by simulating increased load.
- Understand the relationship between virtualization, load balancing, and elasticity.

---

**Theory**

Load balancing is the practice of distributing incoming network traffic across multiple target resources, such as EC2 instances or containers. This distribution ensures high availability and fault tolerance by preventing any single resource from becoming overwhelmed. AWS offers several load balancer types tailored to different needs. The Application Load Balancer operates at Layer 7 of the OSI model, handling HTTP and HTTPS traffic and supporting advanced routing based on request path or host headers. The Network Load Balancer operates at Layer 4, providing ultra low latency for TCP and UDP traffic. The Gateway Load Balancer is designed for integrating third party virtual appliances into the network path.

Auto Scaling is the mechanism that automatically adjusts the number of EC2 instances in response to changing demand. During periods of high utilization, Auto Scaling adds instances, a process known as scaling out. When demand subsides, it removes unneeded instances, scaling in, to optimize costs. Scaling actions can be governed by policies such as target tracking, which maintains a specified metric threshold like average CPU utilization, step scaling, or scheduled actions.

This dynamic behavior is a direct benefit of cloud virtualization. The ability to provision and terminate server instances programmatically in seconds makes elastic, resilient architectures possible.

---

**Procedure**

**Step 1: Create Security Groups**

**1.1 Create ALB Security Group:**
  - Navigate to **EC2 > Security Groups > Create security group**
  - Set Security group name to `web-alb-sg`
  - Set Description to `Security group for Application Load Balancer`
  - **Inbound rules:**
    - Type: HTTP, Port: 80, Source: 0.0.0.0/0, Description: Allow HTTP from the internet
  - **Outbound rules:** (Keep default - All traffic to 0.0.0.0/0)
  - Click **Create security group**

**1.2 Create Instance Security Group:**
- Navigate to **EC2 > Security Groups > Create security group**
- Set Security group name to `web-instance-sg`
- Set Description to `Security group for web server instances`
- **Inbound rules:**
  - Type: HTTP, Port: 80, Source: Custom -> Select `web-alb-sg`, Description: Allow HTTP from ALB
  - Type: SSH, Port: 22, Source: Anywhere IPv4, Description: SSH access
- **Outbound rules:** (Keep default - All traffic to 0.0.0.0/0)
- Click **Create security group**

**Step 2: Create the Launch Template**
- Navigate to **EC2 > Launch Templates > Create launch template**
- Set Launch template name to `web-server-template`
- Under **Application and OS Images**, select **Amazon Linux 2023 AMI**
- Under **Instance type**, select `t2.micro`
- Under **Key pair**, select an existing key pair or create a new one (recommended for troubleshooting)
- Under **Network settings > Security groups**, select `web-instance-sg`
- Expand **Advanced details**
- In the **User data** field, paste the following script:
  ```bash
  #!/bin/bash
  yum update -y
  yum install httpd -y
  systemctl start httpd
  systemctl enable httpd
  
  # Wait for metadata service
  sleep 10
  INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
  AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
  
  # Create index page
  cat > /var/www/html/index.html << EOF
  <!DOCTYPE html>
  <html>
  <head>
      <title>Load Balancer Test</title>
      <style>
          body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
          .instance { color: #007bff; }
          .az { color: #28a745; }
      </style>
  </head>
  <body>
      <h1>Hello from Instance: <span class="instance">$INSTANCE_ID</span></h1>
      <h2>Availability Zone: <span class="az">$AZ</span></h2>
      <p>Timestamp: $(date)</p>
  </body>
  </html>
  EOF
  
  # Restart httpd to ensure everything is working
  systemctl restart httpd
  ```
- Click **Create launch template**

> Screenshot: Launch template with security group selected [Mandatory].

> Sample:

<img width="1457" height="721" alt="Screenshot 2026-04-16 at 10 32 34 AM" src="https://github.com/user-attachments/assets/7b05e196-395e-42d3-8c61-94021773fc63" />


**Step 3: Create a Target Group**
- Navigate to **EC2 > Load Balancing > Target Groups > Create target group**
- Under **Basic configuration**, choose `Instances` as the target type
- Set Target group name to `web-target-group`
- Set Protocol to `HTTP` and Port to `80`
- Under **Health checks**:
    - Health check protocol: `HTTP`
    - Health check path: `/`
  - **Advanced health check settings:**
    - Healthy threshold: `2`
    - Unhealthy threshold: `2`
    - Timeout: `5 seconds`
    - Interval: `30 seconds`
- Click **Next**, then **Create target group**

> Screenshot: Target group configuration [Mandatory].

> Sample:

<img width="1458" height="641" alt="Screenshot 2026-04-16 at 10 31 34 AM" src="https://github.com/user-attachments/assets/04539088-4094-41c2-af44-f8a1f0a97c5a" />


**Step 4: Create the Application Load Balancer**
- Navigate to **EC2 > Load Balancers > Create load balancer**
- Select `Application Load Balancer` and click **Create**
- Set Load balancer name to `web-alb`
- Set Scheme to `Internet-facing`
- Set IP address type to `IPv4`
- Under **Network mapping**, select your VPC and choose **at least two public subnets** in different AZs
- Under **Security groups**, **remove the default** and select `web-alb-sg`
- Under **Listeners and routing**:
  - Protocol: HTTP, Port: 80
  - Default action: Forward to `web-target-group`
- Click **Create load balancer**

> Screenshot: ALB configuration with security group [Mandatory]

> Sample:

<img width="1460" height="505" alt="Screenshot 2026-04-16 at 10 37 59 AM" src="https://github.com/user-attachments/assets/b819315a-c583-4e63-8469-b1a1d436a7a4" />


**Step 5: Create the Auto Scaling Group**
- Navigate to **EC2 > Auto Scaling > Auto Scaling Groups > Create Auto Scaling group**
- Set Auto Scaling group name to `web-asg`
- Under **Launch template**, select `web-server-template` (latest version) and click **Next**
- Instance type requirements
  - No minimum & No maximum for all.
- Under **Network**:
  - Select your VPC
  - Choose the **same public subnets** you selected for the ALB
  - Click **Next**
- Under **Load balancing**:
  - Select `Attach to an existing load balancer`
  - Choose `Choose from your load balancer target groups`
  - Select `web-target-group`
  - **Health checks:** Select both `EC2` and `ELB` health checks
  - **Health check grace period:** `300 seconds`
  - Click **Next**
- Under **Group size**:
  - Desired capacity: `2`
  - Minimum capacity: `1`
  - Maximum capacity: `4`
- Under **Scaling policies**:
  - Select `Target tracking scaling policy`
  - Metric type: `Average CPU utilization`
  - Target value: `50`
- Click **Next** through remaining options, then **Create Auto Scaling group**

> Screenshot: Auto Scaling Group with health check configuration [Mandatory].

> Sample:

<img width="1458" height="713" alt="Screenshot 2026-04-16 at 11 55 40 AM" src="https://github.com/user-attachments/assets/80747401-c738-4a2f-86ad-332a2f8b101e" />


**Step 6: Verify Initial Instances**
  - **Wait 5-8 minutes** for instances to launch, install software, and pass health checks
  - Navigate to **EC2 > Instances** and confirm two instances with prefix `web-asg` are **Running**
  - Navigate to **Target Groups > web-target-group > Targets tab** and confirm both instances show **Healthy** status
  - If instances show "Unhealthy," wait longer or check the troubleshooting steps below

> Screenshot: Target group showing healthy instances [Mandatory]

**Step 7: Test Load Balancing**
  - Navigate to **EC2 > Load Balancers**
  - Select `web-alb` and copy the **DNS name** from the **Description** tab
  - **Wait 2-3 minutes** after ALB shows "Active" status
  - Open a new browser tab and navigate to: `http://[your-alb-dns-name]`
  - Refresh the page multiple times - you should see different Instance IDs and Availability Zones
  - The page should load without 504 errors

> Screenshot: Browser showing different instance IDs on consecutive refreshes [Mandatory]



**Step 7: Trigger a Scale Out Event**
  - Connect to one of the running instances via SSH.
  - Install the stress tool: `sudo yum install stress -y`.
  - Run a stress test: `stress --cpu 4 --timeout 300s`.
  - Navigate to EC2 > Auto Scaling Groups > `web-asg` > Activity tab.
  - Within a few minutes, observe a new event showing an instance is launching. Also monitor CloudWatch > Alarms to see the "Alarm" state triggered.

**Step 8: Observe Scale In Behavior**
  - Allow the stress test to complete or stop it manually (Ctrl+C).
  - Return to the **Activity** tab of the Auto Scaling group.
  - After a cooldown period of approximately 5 to 10 minutes, observe a **Terminating** event for one of the instances.

**Step 9: Test Self Healing**
  - Navigate to **EC2** > **Instances**.
  - Select one of the instances managed by the Auto Scaling group.
  - Click **Instance state** > **Terminate instance** > **Terminate**.
  - Immediately navigate back to the Auto Scaling group **Activity** tab.
  - Confirm that a new instance launch event is triggered automatically to restore the desired capacity of two.

---

**Results**

The Application Load Balancer successfully distributed web traffic across multiple EC2 instances. The Auto Scaling group maintained the configured desired capacity and automatically scaled out when the simulated CPU load exceeded the 50 percent target threshold. After the load subsided, the group scaled back in, demonstrating cost efficiency. The system also demonstrated self healing behavior by automatically replacing a manually terminated instance.

---

**Discussion and Conclusion**

This lab illustrated two fundamental capabilities of cloud infrastructure that are enabled by virtualization. The Application Load Balancer provided a single point of entry while distributing requests across a pool of instances, eliminating single points of failure and improving responsiveness. Auto Scaling delivered horizontal elasticity by dynamically matching the number of running instances to real time demand. The self healing nature of the Auto Scaling group further enhanced system resilience. The combination of load balancing and auto scaling forms the foundation for designing highly available, scalable, and cost effective applications in the cloud.

---

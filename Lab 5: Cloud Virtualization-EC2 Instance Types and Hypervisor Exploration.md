# Exploring Virtualization Technology with Amazon EC2 and AWS Nitro System

---

**Objectives:**
  - Understand server virtualization as implemented by AWS
  - Launch instances on different hypervisor technologies (Xen vs. Nitro)
  - Explore instance families and their virtualization characteristics
  - Examine hardware virtualization, paravirtualization, and HVM (Hardware Virtual Machine)
  - Monitor instance performance and resource allocation

---
 
**Theory:**

Virtualization forms the backbone of cloud computing. It creates virtual representations of physical resources such as servers, storage, and networks, allowing multiple isolated environments to coexist on a single physical machine.

**Types of Virtualization:**
  - Server Virtualization: Multiple virtual servers on one physical server
  - Storage Virtualization: Pooling physical storage into a single logical unit
  - Network Virtualization: Virtual networks decoupled from physical hardware
  - Desktop Virtualization: Virtual desktops delivered to end users

**Hypervisors manage virtual machines:**
  - Type 1 (Bare-metal): Runs directly on hardware. Examples: AWS Nitro, Xen, VMware ESXi
  - Type 2 (Hosted): Runs on top of an OS. Examples: VirtualBox, VMware Workstation

**AWS Virtualization Evolution:**
  - Xen Hypervisor: Used in earlier instance types (C3, M3, T1). Supports HVM (Hardware Virtual Machine) and PV (Paravirtualized) modes.
  - AWS Nitro System: Custom built hypervisor for newer instances (C5, M5, T3, and beyond). Offloads networking, storage, and security to dedicated Nitro Cards, delivering near bare metal performance.

---

**Procedure:**

Step 1: Open the AWS Management Console and go to the EC2 dashboard
  - Click the Launch instance button.

Step 2: Launch a Nitro-based instance
  - Name: Nitro-Instance
  - Application and OS Images: Select Amazon Linux 2023 AMI.
  - Instance type: t3.micro (Nitro-based, Free Tier eligible)
  - Key pair: Select an existing key pair or create a new one.
  - Keep other settings as default and click Launch instance.

Step 3: Launch a Xen-based instance (if available in your region)
  - Return to the EC2 dashboard and click Launch instance again.
  - Name: Xen-Instance
  - Application and OS Images: Select Amazon Linux 2023 AMI.
  - Instance type: t2.micro (Xen-based, Free Tier eligible).
  - Key pair: Select an existing key pair or create a new one.
  - Click Launch instance.

> Screenshot: Both instances running in the EC2 dashboard [Mandatory].

> Sample:

<img width="1470" height="259" alt="Screenshot 2026-04-15 at 7 54 43 PM" src="https://github.com/user-attachments/assets/069c0485-f691-44db-bad8-9358a5fb3494" />


Step 4: Verify Hypervisor and System Information

> Install the required package `sudo yum install -y dmidecode`

- Connect to Nitro-Instance via SSH.
  - Run the following commands and note the output:
```bash
lscpu | grep -i "Virtualization"
```
>Referred to as Hardware Virtual Machine (HVM).
```bash
lscpu | grep -i "Hypervisor vendor"
```
>Nitro uses KVM as the hypervisor.
```bash
sudo dmidecode -s system-manufacturer                         
```
>All Nitro instances report Amazon EC2.

> Screenshot: Terminal output showing hypervisor information from the instance [Mandatory].

> Sample:

<img width="821" height="233" alt="Screenshot 2026-04-16 at 6 19 53 AM" src="https://github.com/user-attachments/assets/4620b21c-e98d-443b-8fec-3909d48e3428" />


- Connect to Xen-Instance via SSH.
  - Run the following commands and note the output:
```bash
lscpu | grep -i "Virtualization"
```
>Referred to as Hardware Virtual Machine (HVM).
```bash
lscpu | grep -i "Hypervisor vendor"
```
>Xen uses Xen as the hypervisor.
```bash
sudo dmidecode -s system-manufacturer                         
```
>All Xen instances report Xen.

> Screenshot: Terminal output showing hypervisor information from the instance [Mandatory].

> Sample:

<img width="897" height="189" alt="Screenshot 2026-04-16 at 6 42 39 AM" src="https://github.com/user-attachments/assets/b4255dfb-ffc3-4261-a3d1-9af4019b4d68" />



Step 5: Compare Network Drivers

>List all network interfaces (look for 'ens', 'eth', or 'enX'): `ip -br link show`

  - On Nitro-Instance, run: `ethtool -i eXXX`.
    - Example:
      
<img width="697" height="247" alt="Screenshot 2026-04-16 at 6 52 56 AM" src="https://github.com/user-attachments/assets/18741229-ee87-4aa1-9f1c-abe00a9e3a2d" />

      
  - On Xen-Instance, run: `ethtool -i eXXX`. Observe the legacy driver name.
    - Example:
      
<img width="682" height="246" alt="Screenshot 2026-04-16 at 6 50 48 AM" src="https://github.com/user-attachments/assets/9505a7eb-a4b1-4068-afc7-65ff1bb49f13" />


Step 6: Explore Instance Families and their purposes

  - In the EC2 console navigation pane, click Instance types.
  - Use the filter bar to view:
    
    - General Purpose (T3, M5): Balanced compute, memory, networking
      - <img width="1449" height="603" alt="Screenshot 2026-04-16 at 6 56 46 AM" src="https://github.com/user-attachments/assets/deb0a94b-f71b-4bac-928b-819ea5e6bd8d" />
      
    - Compute Optimized (C5): CPU-intensive workloads
      - <img width="1457" height="660" alt="Screenshot 2026-04-16 at 6 58 07 AM" src="https://github.com/user-attachments/assets/79b91311-5ec1-4d78-a2b3-a73c296fe1a9" />

 
    - Memory Optimized (R5): Large in-memory datasets
      -  <img width="1454" height="602" alt="Screenshot 2026-04-16 at 7 02 06 AM" src="https://github.com/user-attachments/assets/ef368545-3ec0-4b9b-b38c-34655265ead3" />

    
    - Storage Optimized (I3): High sequential read/write
      - <img width="1455" height="602" alt="Screenshot 2026-04-16 at 7 02 32 AM" src="https://github.com/user-attachments/assets/c26e98b3-eae0-487d-b246-6ca0ca586b54" />

    
    - Accelerated Computing (P3, G4): GPU for ML/AI
      -  <img width="1470" height="565" alt="Screenshot 2026-04-16 at 7 03 13 AM" src="https://github.com/user-attachments/assets/37ff2718-5581-490d-8b33-06d9fd47a50d" />
   

Step 7: Monitor virtualized resource usage with CloudWatch
  - Navigate to the CloudWatch service
  - Click Metrics in the left menu, then metrics > All metrics > EC2: View automatic dashboard > Per-Instance Metrics
  - View: `CPUUtilization`, `NetworkIn`, `NetworkOut`
  - Switch to the Graphed metrics tab to view the current utilization.
    
>Screenshot: CloudWatch dashboard with EC2 instance metrics [Mandatory].

>Sample:

<img width="1458" height="695" alt="Screenshot 2026-04-16 at 7 13 18 AM" src="https://github.com/user-attachments/assets/1586ed40-5ef6-4d4f-83b3-caaa446b583d" />


Step 8: Perform a CPU Stress Test
  - Connect via SSH to either of the running instances.
  - Install the `stress` utility:
    ```bash
    sudo yum install stress -y
    ```
  - Run a stress test on 2 CPU cores for 60 seconds:
    ```bash
    stress --cpu 2 --timeout 60s
    ```
  - Return to the CloudWatch graph and refresh it after a minute to observe the spike in CPU utilization.

>Screenshot: CloudWatch showing CPU spike during stress test [Mandatory].

>Sample:

<img width="1459" height="654" alt="Screenshot 2026-04-16 at 7 28 35 AM" src="https://github.com/user-attachments/assets/3872449c-34b5-4fd7-bf4d-7aabd62939f3" />


Step 9: Examine Placement Groups:
  - Return to the EC2 console.
  - In the navigation pane, Network & Security > Placement Groups.
  - Click Create placement group.
    - `Cluster`: `Low latency` in a single AZ
    - `Spread`: Each instance on different hardware
    - `Partition`: Distributed across logical partitions
  - Read the description to understand how each strategy distributes instances across underlying hardware. (You do not need to create a group to complete this step.) 

>Screenshot: Placement group creation options [Mandatory].

>Sample:

<img width="1129" height="419" alt="Screenshot 2026-04-16 at 7 36 50 AM" src="https://github.com/user-attachments/assets/8367393f-fbb7-4585-883f-ffcd8fd0fad7" />


---

**Results:**
  - Successfully launched Nitro-based (t3.micro) and Xen-based (t2.micro) instances
  - Identified hypervisor types and network driver differences
  - Explored instance families designed for different workload types
  - Monitored virtualized resource allocation using CloudWatch
  - Observed CPU utilization behavior during stress testing

---

**Discussion and Conclusion:**

This lab explored AWS's virtualization implementation. AWS has evolved from the Xen hypervisor to the custom Nitro System, which offloads virtualization functions to dedicated hardware, achieving near bare metal performance. The Nitro hypervisor is a lightweight Type 1 hypervisor that provides strong isolation between instances. Comparing t2 (Xen) and t3 (Nitro) instances revealed differences in network drivers (ENA vs. legacy) and performance characteristics. Instance families demonstrate how virtualization allows the same physical hardware to be partitioned and optimized for different workload profiles. CloudWatch provides visibility into virtualized resource consumption, essential for capacity planning and performance management.

---

# Title: Exploring Virtualization Technology with Amazon EC2 and AWS Nitro System

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
  
> Screenshot: Instance type selection showing t3.micro


Step 3: Launch a Xen-based instance (if available in your region)
  - Return to the EC2 dashboard and click Launch instance again.
  - Name: Xen-Instance
  - Application and OS Images: Select Amazon Linux 2023 AMI.
  - Instance type: t2.micro (Xen-based, Free Tier eligible).
  - Key pair: Select an existing key pair or create a new one.
  - Click Launch instance.

> Screenshot: Both instances running in the EC2 dashboard


Step 4: Verify Hypervisor and System Information

  - Connect to Nitro-Instance via SSH.
  - Run the following commands and note the output:
```bash
curl http://169.254.169.254/latest/meta-data/instance-type      # Check virtualization type

cat /sys/hypervisor/type                                        # Shows "xen" for Xen-based instances
```
```bash
# Check CPU information
lscpu                                                          # Look for: Hypervisor vendor, Virtualization type
```
```bash
# Check DMI information (Nitro instances)
sudo dmidecode -s system-manufacturer                          # Shows "Amazon EC2" for Nitro instances
```
```bash
# Check system info
uname -a
cat /proc/cpuinfo
```

> Screenshot: Terminal output showing hypervisor information from the instance.

- Connect to Xen-Instance via SSH.
- Run the following commands and note the output:
  - `cat /sys/hypervisor/type`
  - `lscpu`
  - `lscpu | grep -i hypervisor`

> Screenshot: Terminal output showing hypervisor information from the instance.


Step 5: Compare Network Drivers
  - On Nitro-Instance, run: `ethtool -i eth0`. Verify the driver is `ena`.
  - On Xen-Instance, run: `ethtool -i eth0`. Observe the legacy driver name.

> Screenshot: Network driver comparison between Nitro and Xen instances.


Step 6: Explore Instance Families and their purposes

  - In the EC2 console navigation pane, click Instance types.
  - Use the filter bar to view:
    - General Purpose (T3, M5): Balanced compute, memory, networking
    - Compute Optimized (C5): CPU-intensive workloads
    - Memory Optimized (R5): Large in-memory datasets
    - Storage Optimized (I3): High sequential read/write
    - Accelerated Computing (P3, G4): GPU for ML/AI
    - Review the column for Hypervisor to see which families use `nitro` versus `xen`

> Screenshot: EC2 instance types page showing family comparison

Step 7: Monitor virtualized resource usage with CloudWatch
  - Navigate to the CloudWatch service
  - Click Metrics in the left menu, then All metrics
  - Browse to EC2 > Per-Instance Metrics
  - View: `CPUUtilization`, `NetworkIn`, `NetworkOut`, `DiskReadOps`
  - Switch to the Graphed metrics tab to view the current utilization.
>Screenshot: CloudWatch dashboard with EC2 instance metrics

Step 8: Perform a CPU Stress Test
  - Connect via SSH to either of the running Linux instances.
  - Install the `stress` utility:
    ```bash
    sudo yum install stress -y
    ```
  - Run a stress test on 2 CPU cores for 60 seconds:
    ```bash
    stress --cpu 2 --timeout 60s
    ```
  - Immediately return to the CloudWatch graph and refresh it after a minute to observe the spike in CPU utilization.

>Screenshot: CloudWatch showing CPU spike during stress test

Step 9: Examine Placement Groups (virtualization-related):
  - Return to the EC2 console.
  - In the navigation pane, click Placement Groups.
  - Click Create placement group.
    - Cluster: Low-latency in a single AZ
    - Spread: Each instance on different hardware
    - Partition: Distributed across logical partitions
  - Read the description to understand how each strategy distributes instances across underlying hardware. (You do not need to create a group to complete this step.) 

>Screenshot: Placement group creation options

---

**Results:**
  - Successfully launched Nitro-based (t3.micro) and Xen-based (t2.micro) instances
  - Identified hypervisor types and network driver differences
  - Explored instance families designed for different workload types
  - Monitored virtualized resource allocation using CloudWatch
  - Observed CPU utilization behavior during stress testing

---

**Discussion and Conclusion:**

This lab explored AWS's virtualization implementation. AWS has evolved from the Xen hypervisor to the custom Nitro System, which offloads virtualization functions to dedicated hardware, achieving near bare-metal performance. The Nitro hypervisor is a lightweight Type-1 hypervisor that provides strong isolation between instances. Comparing t2 (Xen) and t3 (Nitro) instances revealed differences in network drivers (ENA vs. legacy) and performance characteristics. Instance families demonstrate how virtualization allows the same physical hardware to be partitioned and optimized for different workload profiles. CloudWatch provides visibility into virtualized resource consumption, essential for capacity planning and performance management.

---

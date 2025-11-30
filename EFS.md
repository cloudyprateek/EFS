#  AWS Project — Shared File Storage Amazon EFS  

Amazon EFS (Elastic File System) is a fully managed, scalable, shared file storage service for EC2 instances.  
In this project, I implemented a **shared storage architecture** where multiple EC2 instances access the same file system using **EFS + Mount Targets + Security Groups**.

---

#  Introduction

In many web applications, we need **shared storage** accessible from multiple EC2 servers at the same time.  
EFS provides:

- Auto scalability  
- Shared access  
- Pay-as-you-go model  
- No storage provisioning  
- High availability across AZs  

This project demonstrates how I configured EFS in a real-world scenario.

---

#  Project Objective

. Create a shared file storage system  
. Allow multiple EC2 instances to read/write the same files  
. Configure EFS with mount targets  
. Secure the architecture with proper Security Groups  
. Test the setup professionally  

---

#  Key AWS Concepts Used

###  **EC2 (Elastic Compute Cloud)**  
Virtual machine instances where applications run.

###  **VPC & Subnets**  
Networking layer for EC2 + EFS mount targets.

###  **EFS (Elastic File System)**  
Shared file system accessed by multiple EC2 instances.

###  **Security Groups**  
- SG-EC2 → Allow outbound  
- SG-EFS → Allow inbound NFS (2049)

###  **Mount Targets**  
Endpoints that allow EC2 to mount EFS inside the same VPC.
---
#  Implementation Steps (GUI)

##  **1. Create VPC & Subnets**
- Go to **VPC Console → Create VPC**
- Name: `Shared-VPC`
- Create **2 public subnets** in different AZs  
  Example:  
  - `public-subnet-a`  
  - `public-subnet-b`
- Attach Internet Gateway
- Update Route Table → Add IGW route
---

##  **2. Create Security Groups**

### **SG for EC2**
- Name: SG-EC2  
- Inbound: SSH (22) from your IP  
- Outbound: All traffic allowed  

### **SG for EFS**
- Name: SG-EFS  
- Inbound: NFS (2049) from SG-EC2  
- Outbound: All traffic allowed  

---

##  **3. Launch EC2 Instances (GUI Steps)**
- Go to **EC2 Console → Launch Instance**  
- AMI: Amazon Linux 2  
- Type: t2.micro (Free Tier)  
- Network: Use the VPC created earlier  
- Subnet: public-subnet-a  
- Security Group: SG-EC2  

_Create the second EC2 in public-subnet-b with the same steps._

---

##  **4. Create an EFS File System**
- Go to **EFS Console → Create EFS**  
- Name: `shared-efs`  
- VPC: Select your VPC  
- Enable mount targets in all subnets  
- Attach Security Group: **SG-EFS**  

---

##  **5. Install NFS Utils on EC2 (CLI)**

SSH into both EC2 instances and run:

```bash
sudo yum update -y
sudo yum install -y amazon-efs-utils
sudo yum install -y nfs-utils
```
---
##  6. Create Mount Directory
```bash
sudo mkdir /mnt/efs1 (For First EC2 Instance)
sudo mkdir /mnt/efs2 (for Second EC2 Instance)
```
---
## 7. Mount EFS to EC2 (both instance)
```bash
sudo mount -t efs <file-system-id>:/ /mnt/efs1
sudo mount -t efs <file-system-id>:/ /mnt/efs2
```
---
## 8. Testing Setup
On EC2 Instance A:
cd /mnt/efs1
echo "Hello from Instance A" | sudo tee testfile.txt

On EC2 Instance B:
cd /mnt/efs2
cat testfile.txt

If the file appears → SHARED EFS SUCCESSFULLY CONFIGURED
---
## Conclusion

I successfully created a scalable shared file storage architecture using Amazon EFS.
This setup is ideal for:

Web applications

Shared logs

CMS systems

Microservices needing common storage

---



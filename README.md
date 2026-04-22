# 🚀 AWS CLI Lab & Documentation (Step-by-Step)

## 📘 Basic Definitions

### 🔹 What is CLI?

A **Command Line Interface (CLI)** is a text-based interface where you interact with a system by typing commands instead of using a graphical interface.

**Example:**

```
ls
```

This command lists files in a directory.

------

### 🔹 What is AWS CLI?

The **AWS CLI (Amazon Web Services Command Line Interface)** is a tool that allows you to manage AWS services directly from your terminal using commands.

It interacts with services like:

- Amazon EC2
- Amazon S3
- Amazon VPC

**Example:**

```
aws s3 ls
```

Lists all S3 buckets in your account.

------

### 🔹 How do we create resources (like VPC) using AWS CLI?

You use AWS CLI commands that directly call AWS APIs.

**Example:**

```
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

👉 This creates a Virtual Private Cloud (VPC) with a defined IP range.

------

# 🖥️ Part 1: Launch and Prepare EC2 Instance

### Step 1: Launch EC2 Instance

- Go to AWS Console
- Launch a basic instance (Ubuntu recommended)
- Connect using SSH

![preview](./EC2-server%20Launch.png)

------

### Step 2: Update the Server

```
sudo apt -y update
```

👉 Updates package lists.

------

### Step 3: Install unzip

```
sudo apt install unzip -y
```

👉 Needed to extract AWS CLI package.

------

### Step 4: Install AWS CLI

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
```

👉 Installs AWS CLI on your EC2 instance.

![preview](./Installing%20AWS-CLI%20in%20linux%20OS.png)

------

# 🔐 Part 2: IAM Role Setup

### Step 1: Create IAM Role

- Go to IAM → Roles → Create Role
- Use Case: EC2
- Attach required policy (e.g., S3 Full Access for testing)
- Give role name → Create

👉 IAM Role allows EC2 to access AWS services securely without credentials.

![preview](./Creation%20of%20role.png)

------

### Step 2: Attach Role to EC2

- Go to EC2 → Instance
- Actions → Security → Modify IAM Role
- Attach created role

![preview](./Modifying%20role.png)

------

# ☁️ Part 3: Verify AWS CLI Connection

```
aws s3 ls
```

👉 Shows all S3 buckets → confirms CLI is working.

![preview](./AWS-CLI-Connected.png)

------

# 📦 Part 4: Working with S3

### 🔹 Create Bucket

```
aws s3 mb s3://cli-satwik-bucket-2
```

👉 `mb` = Make Bucket

![preview](./Creating%20s3%20bucket%20by%20using%20cli%20commands.png)

![preview](./Bucket%20was%20created%20by%20CLI%20command.png)

------

### 🔹 Upload File

```
aws s3 cp awscliv2.zip s3://cli-satwik-bucket-2
```

👉 `cp` = Copy file to S3

![preview](./Uploading%20zip%20file%20or%20anything%20by%20CLI%20Command.png)

![preview](./Uploaded%20file%20in%20s3%20bucket.png)

------

# 🌐 Part 5: Create VPC Using AWS CLI

## 🔹 Step 1: Create VPC

```
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

👉 CIDR defines IP range for VPC.

![preview](./VPC-Creation.png)

![preview](./VPC-Created%20seen%20in%20aws%20cli.png)


------

## 🔹 Step 2: Tag VPC

```
aws ec2 create-tags \
--resources vpc-xxxx \
--tags Key=Name,Value=Satwik-VPC
```

👉 Adds name to VPC.

![preview](./Tag%20changed%20in%20VPC%20by%20CLI%20Command.png)


------

## 🔹 Step 3: View VPCs

```
aws ec2 describe-vpcs
```

![preview](./Describing%20VPC.png)


------

# 🌍 Part 6: Create Subnets

### Public Subnets

```
aws ec2 create-subnet \
--vpc-id vpc-xxxx \
--cidr-block 10.0.1.0/24 \
--availability-zone us-east-1a
aws ec2 create-subnet \
--vpc-id vpc-xxxx \
--cidr-block 10.0.2.0/24 \
--availability-zone us-east-1b
```

![preview](./Subnets%20Created%20by%20using%20CLI%20commads.png)

![preview](./Associated%20Private%20Subnets.png)


------

### Private Subnets

```
aws ec2 create-subnet \
--vpc-id vpc-xxxx \
--cidr-block 10.0.3.0/24 \
--availability-zone us-east-1a
aws ec2 create-subnet \
--vpc-id vpc-xxxx \
--cidr-block 10.0.4.0/24 \
--availability-zone us-east-1b
```

👉 Public = internet access
 👉 Private = no direct internet access


------

# 🌐 Part 7: Internet Gateway (IGW)

### Create IGW

```
aws ec2 create-internet-gateway
```

![preview](./Creating%20Internetgateway.png)

------

### Attach IGW to VPC

```
aws ec2 attach-internet-gateway \
--internet-gateway-id igw-xxxx \
--vpc-id vpc-xxxx
```

![preview](./Attaching%20IGW%20to%20VPC.png)


------

# 🛣️ Part 8: Route Tables

### Public Route Table

```
aws ec2 create-route-table --vpc-id vpc-xxxx
```

![preview](./Creating%20Route%20Tables.png)

------

### Add Internet Route

```
aws ec2 create-route \
--route-table-id rtb-xxxx \
--destination-cidr-block 0.0.0.0/0 \
--gateway-id igw-xxxx
```

![preview](./Adding%20Internet%20to%20Route%20Table.png)


------

### Associate with Public Subnets

```
aws ec2 associate-route-table \
--subnet-id subnet-xxxx \
--route-table-id rtb-xxxx
```

![preview](./Associating%20Route%20tables%20with%20subnets%20(Public).png)

------

### Private Route Table

```
aws ec2 create-route-table --vpc-id vpc-xxxx
```

👉 No internet route added.

![preview](./Creating%20Private%20Route%20Table.png)


------

# 🔒 Part 9: Network ACL (NACL)

### Create NACL

```
aws ec2 create-network-acl --vpc-id vpc-xxxx
```
![preview](./Create%20Network%20ACL%20(NACL).png)

![preview](./NACL%20Created%20in%20AWS-CLI.png)

------

### Inbound Rule (Allow HTTP)

```
aws ec2 create-network-acl-entry \
--network-acl-id acl-xxxx \
--rule-number 100 \
--protocol tcp \
--port-range From=80,To=80 \
--cidr-block 0.0.0.0/0 \
--rule-action allow \
--ingress
```

------

### Outbound Rule

```
aws ec2 create-network-acl-entry \
--network-acl-id acl-xxxx \
--rule-number 100 \
--protocol -1 \
--cidr-block 0.0.0.0/0 \
--rule-action allow \
--egress
```

------

# 🛡️ Part 10: Security Group (SG)

### Create Security Group

```
aws ec2 create-security-group \
--group-name my-sg \
--description "My Security Group" \
--vpc-id vpc-xxxx
```
![preview](./Creating%20Security%20Group.png)

![preview](./Security%20Group%20Creation%20seen%20in%20AWS%20-%20CLI.png)


------

### Allow HTTP

```
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxx \
--protocol tcp \
--port 80 \
--cidr 0.0.0.0/0
```
![preview](./Add%20inbound%20rule%20(HTTP).png)


------

### Allow SSH

```
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxx \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0
```
![preview](./Add%20SSH%20access.png)


------

# 📌 Summary

By completing this lab, you have:

- Installed AWS CLI
- Connected EC2 to AWS services
- Created and managed S3 buckets
- Built a full VPC network using CLI
- Configured:
  - Subnets
  - Internet Gateway
  - Route Tables
  - NACL
  - Security Groups

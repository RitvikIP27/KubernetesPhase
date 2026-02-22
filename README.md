---
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:ff7a18,100:0f0f0f&height=200&section=header&text=Kubernetes%20From%20Scratch&fontSize=35&fontColor=ffffff"/>
</p></h1>

<p align="center">
  Building Kubernetes from scratch | kOps | AWS | Cluster Internals
</p>


<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-v1.28-blue?style=for-the-badge&logo=kubernetes&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazonaws&logoColor=white"/>
  <img src="https://img.shields.io/badge/kOps-Cluster%20Provisioning-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Linux-Ubuntu%2024.04-red?style=for-the-badge&logo=ubuntu&logoColor=white"/>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/RitvikIP27/KubernetesPhase?style=social"/>
  <img src="https://img.shields.io/github/forks/RitvikIP27/KubernetesPhase?style=social"/>
  <img src="https://img.shields.io/github/issues/RitvikIP27/KubernetesPhase"/>

</p>

---

## 📘 About This Repository

This repository is created with the intent to document my work with **Kubernetes**.  
It walks through installation, cluster creation using kOps, architecture understanding, and validation steps.

Work in progress 🚧


## 🏗 Kubernetes Architecture – Control Plane & Data Plane
<p align="center">
  <img src="kuBEARCH.png" width="800"/>
</p>


---

### 🎛 Control Plane Components

- **API Server** – The central entry point of the cluster that receives and validates all requests and communicates with other control plane components.

- **Scheduler** – Assigns newly created Pods to suitable Worker Nodes based on resource availability and constraints.

- **etcd** – A distributed key-value store that persistently stores all cluster configuration and state data.

---

### ⚙ Data Plane (Worker Node) Components

- **kubectl** – Command-line tool used by users to interact with the Kubernetes API Server.

- **Container Runtime** – Responsible for pulling container images and running containers (e.g., containerd, CRI-O).

- **kube-proxy** – Maintains network rules on nodes and enables communication between Pods and Services.

---

### 🔄 High-Level Flow

1. User sends request using `kubectl`.
2. API Server validates and processes the request.
3. Scheduler assigns the Pod to a Worker Node.
4. Container Runtime pulls the image and starts the container.
5. kube-proxy manages networking and service routing.
6. etcd stores the cluster state persistently.

---


# 🚀 Kubernetes Setup on Ubuntu 24.04 (kOps + AWS)

This guide walks through installing and configuring:

- ☸️ `kubectl`
- ☁️ `AWS CLI`
- 🏗 `kOps`
- 🔐 Secure Kubernetes APT Repository

Tested on **Ubuntu 24.04 LTS**

---

## 📦 1️⃣ Update System & Install Base Dependencies

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl apt-transport-https
```

### 🔎 Explanation

| Command | Purpose |
|----------|----------|
| `apt-get update` | Refreshes local package index |
| `ca-certificates` | Enables trusted HTTPS connections |
| `curl` | CLI tool to download files |
| `apt-transport-https` | Allows APT to use HTTPS repositories |

---

## 🔐 2️⃣ Add Kubernetes Repository (Secure Method)

### Download & Configure GPG Key

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### 🔎 Why This Is Required

- Downloads Kubernetes repository signing key
- Converts it to binary format (`--dearmor`)
- Ensures APT installs only trusted packages
- Prevents MITM or tampered package installations

---

### Add Kubernetes APT Repository

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | \
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

This creates:

```
/etc/apt/sources.list.d/kubernetes.list
```

---

## ☸️ 3️⃣ Install kubectl

```bash
sudo apt-get update
sudo apt-get install -y kubectl
```

### 🔎 What is kubectl?

`kubectl` is the Kubernetes command-line client used to:

- Communicate with Kubernetes API server
- Deploy workloads
- Inspect cluster resources
- Manage pods, services, nodes

---

## ☁️ 4️⃣ Install AWS CLI

```bash
sudo snap install aws-cli --classic
```

### 🔎 Why Snap?

Ubuntu 24.04 prefers Snap for some packages.

`--classic` allows AWS CLI full system access.

---

### Add AWS CLI to PATH (if needed)

```bash
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

To make permanent:

```bash
echo 'export PATH="$PATH:/home/ubuntu/.local/bin/"' >> ~/.bashrc
source ~/.bashrc
```

---

## 🏗 5️⃣ Install kOps

kOps is used to create and manage production-grade Kubernetes clusters on AWS.

---

### Download Latest kOps Binary

```bash
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
```

### 🔎 What This Does

- Fetches latest release dynamically using GitHub API
- Downloads Linux AMD64 binary

---

### Make It Executable

```bash
chmod +x kops-linux-amd64
```

---

### Move to System Path

```bash
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

Verify installation:

```bash
kops version
```

---

# 🧠 Summary

After completing these steps, you now have:

| Tool | Purpose |
|------|----------|
| ☸️ kubectl | Interact with Kubernetes cluster |
| ☁️ AWS CLI | Manage AWS infrastructure |
| 🏗 kOps | Provision Kubernetes cluster on AWS |
| 🔐 Secure APT repo | Verified Kubernetes package installation |

---

# 🚀 Next Steps

1. Configure AWS credentials:

```bash
aws configure
```

2. Create S3 bucket for kOps state store.

3. Create Kubernetes cluster using kOps.

---

# 📌 Notes

- Ensure IAM user has required permissions.
- Always use signed repositories.
- Keep Kubernetes version updated.
- Destroy infrastructure when finished to avoid AWS billing charges.



## 🗂️ Kubernetes Cluster Installation Using kOps

Please follow the steps carefully and read each command before executing.

---

### 🪣 Step 1: Create an S3 Bucket for kOps State Store

kOps uses an S3 bucket to store cluster configuration and state files.

```bash
aws s3api create-bucket \
  --bucket kops-abhi-storage \
  --region us-east-1
```

> ⚠️ If you are creating the bucket outside `us-east-1`, you must specify:
>
> ```bash
> --create-bucket-configuration LocationConstraint=<your-region>
> ```

Example for Mumbai region:

```bash
aws s3api create-bucket \
  --bucket kops-abhi-storage \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```

---

### 🌍 Step 2: Export kOps State Store Environment Variable

To avoid passing `--state` flag every time:

```bash
export KOPS_STATE_STORE=s3://kops-abhi-storage
```

To make it permanent:

```bash
echo 'export KOPS_STATE_STORE=s3://kops-abhi-storage' >> ~/.bashrc
source ~/.bashrc
```

---

### ☸️ Step 3: Create the Kubernetes Cluster Configuration

```bash
kops create cluster \
  --name=demok8scluster.k8s.local \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=8 \
  --node-volume-size=8
```

### 🔎 What This Command Does

- `--name` → Cluster DNS name
- `--zones` → AWS Availability Zone
- `--node-count` → Number of worker nodes
- `--node-size` → EC2 instance type for workers
- `--master-size` → EC2 instance type for control plane
- `--master-volume-size` → EBS size for master
- `--node-volume-size` → EBS size for worker

> ⚠️ This command only creates cluster configuration — it does NOT create resources yet.

---

### ✏️ Step 4: Edit Cluster Configuration (Optional but Recommended)

Before creating infrastructure:

```bash
kops edit cluster demok8scluster.k8s.local
```

You can modify:

- Instance types
- Networking configuration
- Kubernetes version
- Topology (public/private)

Save and exit the editor.

---

### 🏗️ Step 5: Build the Cluster (Provision Infrastructure)

Now actually create AWS resources:

```bash
kops update cluster demok8scluster.k8s.local --yes
```

### 🔎 What Happens Internally

kOps creates:

- EC2 instances (Master + Worker)
- Auto Scaling Groups
- IAM Roles
- Security Groups
- Elastic Load Balancer
- Route53 records (if configured)
- VPC resources (if required)

⏳ This process takes several minutes.

---

### ✅ Step 6: Validate the Cluster

After a few minutes:

```bash
kops validate cluster demok8scluster.k8s.local
```

---

## 🧹 Destroy the Cluster (Important to Avoid AWS Charges)

When finished, delete the cluster safely:

```bash
kops delete cluster demok8scluster.k8s.local --yes
```

If needed, remove S3 bucket:

```bash
aws s3 rb s3://kops-abhi-storage --force
```

---



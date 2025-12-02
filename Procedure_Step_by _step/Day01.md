Great — here is your **Day 1 Documentation** in clean, simple, interview-friendly, and GitHub-ready format.

You can copy this file as:

📁 **procedure-from-scratch/day1.md**
or
📄 **Day1_AWS_Git_Setup.md**

---

# ✅ **Day 1 – AWS & Git Setup + EC2 Launch & SSH Access**

## 🎯 **Objective**

Set up the foundation required for the end-to-end DevOps CI/CD project by configuring Git, AWS IAM, AWS CLI, and launching an EC2 instance with SSH connectivity.

---

# 📝 **1. Installed & Configured Git**

### **Steps Performed**

* Downloaded & installed Git on Windows.
* Verified installation:

  ```bash
  git --version
  ```
* Created a GitHub repository:
  **ultimate-DevOps-ci-cd-project**
* Cloned project locally and updated README.

### **Outcome**

Git is ready for version control, documentation, and CI/CD integrations.

---

# 📝 **2. Installed AWS CLI**

### **Steps Performed**

* Installed **AWS CLI v2** on Windows.
* Verified installation:

  ```bash
  aws --version
  ```

### **Outcome**

System is ready to connect and interact with AWS services programmatically.

---

# 📝 **3. Created IAM User for DevOps Work**

### **Details**

* Username: **devops-user**
* Permissions: **AdministratorAccess** (project purpose)
* Generated **Access Key & Secret Key** for CLI authentication.

### **Configured AWS CLI**

Using:

```bash
aws configure
```

Provided inputs:

* AWS Access Key
* AWS Secret Key
* Default region: **us-east-1**
* Output format: **json**

### **Verification**

```bash
aws sts get-caller-identity
```

### **Outcome**

IAM user successfully authenticated and ready to operate AWS resources via CLI.

---

# 📝 **4. Created EC2 Key Pair**

### **Details**

* Key pair name: **devops-project-key.pem**
* Moved `.pem` file to:

  ```
  /Desktop/AWS/.ssh
  ```
* Updated permissions:

  ```bash
  chmod 400 devops-project-key.pem
  ```

### **Outcome**

Secure SSH access method created for EC2.

---

# 📝 **5. Launched EC2 Instance**

### **Configuration**

* AMI: **Ubuntu 24.04 LTS**
* Instance Type: **t3.micro (Free-tier eligible)**
* Key Pair: **devops-project-key**
* Security Group:

  * Allow SSH (port 22) from Anywhere (0.0.0.0/0)
* Storage: **8 GB gp3**

### **Outcome**

EC2 instance successfully launched in **us-east-1**.

---

# 📝 **6. Connected to EC2 via SSH**

Using MobaXterm:

```bash
ssh -i "devops-project-key.pem" ubuntu@<PUBLIC-IP>
```

### **Outcome**

Logged into EC2 successfully.
Instance is ready for software installation and upcoming DevOps automation.

---

# 🎉 **Day 1 Completed Successfully**

You now have:

✔ GitHub repo created
✔ AWS CLI configured
✔ IAM user with programmatic access
✔ EC2 key pair & permissions fixed
✔ EC2 instance launched
✔ SSH connection established

You are now fully prepared for **Day 2 – Jenkins Installation & Server Setup**.

---



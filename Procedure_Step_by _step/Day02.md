# 🚀 Day 2 – Tooling Setup on EC2 (Jenkins, Docker, Git, Node.js, Maven, Terraform)

**Project:** Ultimate DevOps CI/CD Project
# Day: 2
# EC2 OS: Ubuntu 24.04 LTS
# Client: Windows laptop using **MobaXterm** for SSH

---

## 🎯 Day 2 Objectives

On Day 2, the focus was to turn our EC2 instance into a **DevOps tools server**:

* Fix SSH/connectivity issues (if any)
* Get **Jenkins** web UI working
* Install and configure:

  * **Git**
  * **Docker** (with proper daemon + permissions)
  * **Docker Compose**
  * **Node.js & npm**
  * **Maven**
  * **Terraform**

Everything below is written so that:

* I can **reproduce** it later
* An interviewer or reviewer can clearly see my **troubleshooting process**

---

## 0️⃣ Starting Point

From Day 1, we already had:

* IAM user: `devops-user` with `AdministratorAccess`
* EC2 instance (Ubuntu 24.04) in region **us-east-1**
* SSH key pair: `devops-project-key.pem`
* Security group with at least **port 22 (SSH)** open
* Basic system updates done previously using:

  ```bash
  sudo apt update -y
  sudo apt upgrade -y
  ```

---

## 1️⃣ Fixing SSH Connectivity (Security Group Issue)

### 🔴 Problem

When trying to SSH into the EC2 instance using MobaXterm, I got:

* **Network error: Connection timed out**

This usually means **security group** or **network** issue, not a key problem.

### ✅ What I Did

1. Opened **AWS Console → EC2 → Instances → Security** tab.
2. Opened the **Security Group** attached to my EC2 instance.
3. In **Inbound rules**, I checked port **22 (SSH)**.
4. I updated the source to:

   * `My IP` (recommended), or
   * `0.0.0.0/0` temporarily for learning (not for production).

After updating the rule, I tried SSH again from MobaXterm using:

* **Protocol:** SSH
* **Host:** `<EC2_PUBLIC_IP>`
* **Username:** `ubuntu`
* **Private key:** `devops-project-key.pem`

✅ SSH connection succeeded.

---

## 2️⃣ Jenkins – Service & Web UI Setup

Jenkins was installed (or installed earlier on Day 2) using:

```bash
# (If not already done)
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update -y
sudo apt install -y fontconfig openjdk-17-jre jenkins
```

### 🔍 Check Jenkins Service

```bash
sudo systemctl status jenkins
```

Expected: `active (running)`

---

### 🔴 Problem – Jenkins UI not reachable on port 8080

Trying to open:

```text
http://<EC2_PUBLIC_IP>:8080
```

gave **“This site can’t be reached / ERR_CONNECTION_TIMED_OUT”**.

### 🧠 Root Cause

* Jenkins was running on the server, but:
* **Security Group** did **not** allow inbound traffic on **port 8080**.

### ✅ Fix

1. Go to **EC2 → Security Groups → (Instance SG) → Inbound rules**.
2. Add rule:

   * **Type:** Custom TCP
   * **Port:** `8080`
   * **Source:** `My IP` (or `0.0.0.0/0` for testing only)

After adding the rule, reloaded:

```text
http://<EC2_PUBLIC_IP>:8080
```

✅ Jenkins welcome page loaded.

---

### 🔐 Unlocking Jenkins

Jenkins asked for the initial admin password from:

```text
/var/lib/jenkins/secrets/initialAdminPassword
```

Command used:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

* Copied the password from the terminal.
* Pasted into the Jenkins unlock screen.
* Clicked **Continue**.

---

### ⚙️ Jenkins Initial Setup

1. Selected **“Install suggested plugins”**.
2. Waited for plugins to finish installing.
3. Created the first admin user (username/password).
4. Confirmed Jenkins URL (default `http://<EC2_PUBLIC_IP>:8080/`).
5. Reached the **Jenkins Dashboard**.

✅ Jenkins is now **ready** for creating jobs and pipelines.

---

## 3️⃣ Git Installation (Inside EC2)

Git is needed so Jenkins and local automation can work with repositories.

### Commands

```bash
sudo apt install git -y
git --version
```

Output:

```text
git version 2.43.0
```

✅ Git installed successfully, no issues.

---

## 4️⃣ Docker Installation & Troubleshooting

We want Docker so we can run containers (React app, tools, etc.).

### 4.1 First Attempt: `apt install docker.io` Stuck at 94%

Command:

```bash
sudo apt install docker.io -y
```

🔴 **Problem**

* Installation progress got stuck at **94%** for a long time.
* Ctrl + C did not properly exit.

This is usually a **dpkg / apt lock** or interrupted installation.

### ✅ Fix – Repair dpkg & APT

Ran the following sequence:

```bash
sudo pkill dpkg                       # kill stuck dpkg if any
sudo dpkg --configure -a             # reconfigure any half-installed packages
sudo apt --fix-broken install -y     # fix broken dependencies
```

This cleaned up the partially installed packages.

---

### 4.2 Install Docker Using Official Script

To get a clean Docker Engine install, I used Docker’s official script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

This installed **Docker Engine - Community**.

---

### 4.3 Docker Daemon Not Running Error

When checking Docker version / running a container:

```bash
docker --version
docker ps
docker run hello-world
```

I saw:

```text
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

### ✅ Fix – Start & Enable Docker Service

```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

After that, `docker ps` started working **with sudo**, but there was another issue.

---

### 4.4 Permission Denied (Non-root Docker Usage)

When running:

```bash
docker run hello-world
```

I saw:

```text
permission denied while trying to connect to the Docker daemon socket
```

This happens because **current user** is not in the `docker` group.

### ✅ Fix – Add User to docker Group

```bash
sudo usermod -aG docker $USER
```

Then I **logged out and back in** (SSH reconnect) so group changes took effect.

✅ After reconnecting:

```bash
docker run hello-world
```

Output:

```text
Hello from Docker!
...
This message shows that your installation appears to be working correctly.
```

Docker is now fully working for the `ubuntu` user.

---

## 5️⃣ Docker Compose Installation & Fix

Initially I tried installing Docker Compose as a standalone binary:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/2.29.2/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

🔴 **Problem**

`docker-compose --version` failed with:

```text
/usr/local/bin/docker-compose: line 1: Not: command not found
```

Meaning: the downloaded file content was not a valid binary (likely HTML or redirect).

### ✅ Correct Approach – Use Plugin Package

On modern Ubuntu + Docker, Docker Compose is installed as a **plugin**:

```bash
sudo apt update
sudo apt install docker-compose-plugin -y
```

Verification:

```bash
docker compose version
```

Output (example):

```text
Docker Compose version v2.5.0
```

✅ Docker Compose is now installed correctly as `docker compose` (not `docker-compose`).

---

## 6️⃣ Node.js & npm Installation

Node.js is required later for running and building the React application.

### Commands

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

Verification:

```bash
node -v
npm -v
```

Output:

```text
v18.20.8
10.8.2
```

✅ Node.js 18 and npm are installed successfully.

---

## 7️⃣ Maven Installation

Maven is needed for Java-based builds (for future backend/microservice parts).

### Install & Verify

```bash
sudo apt install maven -y
mvn -version
```

Output:

```text
Apache Maven 3.8.7
Java version: 17.0.x, vendor: Ubuntu, runtime: /usr/lib/jvm/java-17-openjdk-amd64
```

✅ Maven installation successful.

---

## 8️⃣ Terraform Installation

Terraform will be used later for Infrastructure as Code (IaC).

### 8.1 Add HashiCorp GPG key

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

### 8.2 Add HashiCorp APT Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

### 8.3 Install Terraform

```bash
sudo apt update -y
sudo apt install terraform -y
```

### 8.4 Verify

```bash
terraform version
```

Output:

```text
Terraform v1.4.1
```

✅ Terraform is now installed and ready.

---

## 9️⃣ Final Tooling Validation

At the end of Day 2, I validated all tools with these commands:

```bash
# Jenkins – via browser
# (manually checked http://<EC2_PUBLIC_IP>:8080 is reachable)

git --version
docker --version
docker ps
docker run hello-world
docker compose version
node -v
npm -v
mvn -version
terraform version
```

### ✅ Summary Table

| Tool           | Command                  | Version (example) | Status    |
| -------------- | ------------------------ | ----------------- | --------- |
| Jenkins        | Web UI on `:8080`        | 2.528.2           | ✔ Running |
| Git            | `git --version`          | 2.43.0            | ✔ OK      |
| Docker Engine  | `docker --version`       | 29.1.2            | ✔ OK      |
| Docker Compose | `docker compose version` | v2.5.0            | ✔ OK      |
| Node.js        | `node -v`                | 18.20.8           | ✔ OK      |
| npm            | `npm -v`                 | 10.8.2            | ✔ OK      |
| Maven          | `mvn -version`           | 3.8.7             | ✔ OK      |
| Terraform      | `terraform version`      | 1.4.1             | ✔ OK      |

---

## 🔚 Day 2 – Interview-Style Summary

> On Day 2, I converted an Ubuntu EC2 instance into a full DevOps tool server.
> I configured Jenkins and exposed it securely via AWS Security Groups, then installed Git, Docker, Docker Compose, Node.js, Maven, and Terraform.
> During the process I hit several real-world issues: SSH timeouts due to security group rules, Jenkins port 8080 being blocked, Docker installation hanging at 94% because of dpkg lock, “Cannot connect to Docker daemon” errors, and a broken docker-compose binary. I fixed them by repairing dpkg, installing Docker via the official script, managing services with systemd, updating Linux groups for Docker permissions, and switching to the correct docker-compose plugin package.
> By the end of Day 2, all DevOps tools were installed, validated, and ready for building CI/CD pipelines.

---



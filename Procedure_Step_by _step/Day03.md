## 1️⃣ Day 3 Documentation (Markdown for your repo)

You can create a file like:

`procedure/day3-containerizing-node-app.md`

Paste this:

````markdown
# Day 3 – Containerizing a Node.js App with Docker on AWS EC2

## Objective

On Day 3, I deployed a simple Node.js application on my EC2 instance and then containerized it using Docker.  
The final result: I can access the app in a browser via my EC2 public IP on port **3000**, and the app is running **inside a Docker container**, not directly on the host.

---

## 1. Prepare EC2 & Login

- EC2 instance: Ubuntu 24.04 LTS (t3.micro, us-east-1).
- Connected from my Windows laptop using **MobaXterm** via SSH and `.pem` key.

```bash
ssh -i devops-project-key.pem ubuntu@<EC2-PUBLIC-IP>
````

Checked I was in the home directory:

```bash
pwd
# /home/ubuntu
```

---

## 2. Create a Simple Node.js Application

### 2.1 Create project folder

```bash
mkdir devops-node-app
cd devops-node-app
pwd
# /home/ubuntu/devops-node-app
```

### 2.2 Initialize `package.json`

```bash
npm init -y
```

This created a basic `package.json` with default values.

### 2.3 Install Express

```bash
npm install express
```

### 2.4 Create `index.js`

```bash
nano index.js
```

Content:

```js
const express = require("express");
const app = express();
const port = 3000;

app.get("/", (req, res) => {
  res.send("Hello from Day 3 DevOps Node.js App 🚀");
});

app.listen(port, () => {
  console.log(`App listening on http://localhost:${port}`);
});
```

### 2.5 Test the app directly on the EC2 host

Start the app:

```bash
node index.js
# App listening on http://localhost:3000
```

In another terminal or after stopping, I tested with:

```bash
curl http://localhost:3000
# Hello from Day 3 DevOps Node.js App 🚀
```

At this point, the app worked **without Docker**, just using Node.js on the host.

---

## 3. Install & Verify Docker

Docker was already installed from Day 2, but I verified everything:

```bash
docker --version
docker ps
```

If Docker is not running, the service is started with:

```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

I also confirmed that the **ubuntu** user can run Docker without `sudo` (from the Day 2 configuration where we added `ubuntu` to the `docker` group).

---

## 4. Write the Dockerfile

In `/home/ubuntu/devops-node-app`:

```bash
ls
# index.js  package.json  package-lock.json  node_modules/ ...

nano Dockerfile
```

Content of `Dockerfile`:

```dockerfile
# 1) Use a lightweight Node.js base image
FROM node:18-alpine

# 2) Set working directory inside the container
WORKDIR /app

# 3) Copy only package files first (cache layer for npm install)
COPY package*.json ./

# 4) Install dependencies
RUN npm install

# 5) Copy the rest of the application code
COPY . .

# 6) Expose the port that the app will listen on
EXPOSE 3000

# 7) Default command to start the app
CMD ["node", "index.js"]
```

Why this structure?

* Using `node:18-alpine` keeps the image small.
* Copying `package*.json` first allows Docker to cache `npm install` when code changes but dependencies don’t.
* `WORKDIR /app` is the home for our app inside the container.
* `EXPOSE 3000` documents the port our app uses.
* `CMD` defines the default process (Node app) when the container starts.

---

## 5. Build the Docker Image

From the project directory:

```bash
docker build -t devops-node-app:v1 .
```

Docker steps (high level):

* Pulls `node:18-alpine`.
* Copies `package.json` and `package-lock.json`.
* Runs `npm install` inside the image.
* Copies the rest of the code.
* Produces an image tagged `devops-node-app:v1`.

Check the image:

```bash
docker images

# Example output:
# REPOSITORY         TAG   IMAGE ID      SIZE
# devops-node-app    v1    3dd9735e7a09  132MB
# hello-world        latest ...
```

---

## 6. Run the App Inside a Container

Run the container and map port **3000** of the container to port **3000** on the EC2 instance:

```bash
docker run -d \
  --name devops-node-container \
  -p 3000:3000 \
  devops-node-app:v1
```

* `-d` → detached mode.
* `--name` → friendly name for the container.
* `-p 3000:3000` → hostPort:containerPort (exposes app externally).
* `devops-node-app:v1` → image to run.

Check that the container is running:

```bash
docker ps

# Example:
# CONTAINER ID   IMAGE               COMMAND                  STATUS         PORTS
# 298f186203ea   devops-node-app:v1  "docker-entrypoint.s…"   Up ... seconds 0.0.0.0:3000->3000/tcp
```

---

## 7. Test the Containerized App

### 7.1 From EC2 (inside the VM)

```bash
curl http://localhost:3000
# Hello from Day 3 DevOps Node.js App 🚀
```

### 7.2 From my local browser (Windows laptop)

I copied the EC2 **public IP** from the AWS console and opened:

```text
http://<EC2_PUBLIC_IP>:3000
# Example: http://34.231.21.88:3000
```

I successfully saw the message:

> Hello from Day 3 DevOps Node.js App 🚀

This confirmed:

* Security group allows inbound HTTP on port **3000**.
* Port mapping (`-p 3000:3000`) is correct.
* The app is fully running inside Docker, but reachable from the internet.

---

## 8. Summary of What I Learned on Day 3

* How to create a **simple Node.js + Express** application.
* How to **test it locally** on an EC2 instance.
* How to write a **Dockerfile** to containerize the app.
* How Docker layers work (base image → npm install → copy code → CMD).
* How to **build Docker images** and **run containers** with port mapping.
* How to make the containerized app accessible via the **EC2 public IP**.

This sets the foundation for the next steps in the project:

* Pushing images to a container registry (e.g., Docker Hub / ECR).
* Integrating this app into a full CI/CD pipeline using Jenkins, Docker, Terraform, etc.

````





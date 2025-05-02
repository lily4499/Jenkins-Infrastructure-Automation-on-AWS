
# 🚀 Jenkins Infrastructure Automation on AWS with Terraform, NGINX, SSL & Agent Setup

## 🏢 Real-World Scenario

> At **Liliane Tech Solutions**, our DevOps team was tasked with building a secure and scalable CI/CD pipeline to support rapid deployment of containerized applications across multiple environments. Using **Terraform**, we automated the provisioning of AWS EC2 instances for a Jenkins master and agent setup. To ensure secure access, we configured **NGINX as a reverse proxy** and integrated **SSL certificates** via **Certbot**, enabling encrypted communication through a custom domain. By offloading builds to a Jenkins agent and running Dockerized jobs remotely, we reduced build times and improved infrastructure scalability. This setup has become the foundation of our delivery process, enabling teams to ship code faster, safer, and more consistently.

---
## Objective

Setting up a secure, scalable CI/CD infrastructure. You’ll:
- Provision **EC2 instances** via **Terraform**
- Install **Jenkins** with **NGINX reverse proxy**
- Secure access using **Certbot SSL**
- Add a **Jenkins agent** EC2 instance for distributed builds
- Run a **Dockerized pipeline job** on the agent

---

## 📁 Project Structure

```
jenkins-automation/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── provider.tf
│   ├── terraform.tfvars
│   └── outputs.tf
├── nginx/
│   └── jenkins.conf
├── certbot/
│   └── certbot-commands.sh
├── scripts/
│   ├── install_java.sh
│   ├── install_jenkins.sh
│   ├── install_nginx.sh
│   ├── setup_firewall.sh
│   └── run_demo_job.Jenkinsfile
```

---

## ✅ Step-by-Step Breakdown

### 1. 🌐 Provision EC2 Instances with Terraform

#### `provider.tf`
```hcl
provider "aws" {
  region = "us-east-1"
}
```

#### `main.tf` (example)
```hcl
resource "aws_instance" "jenkins_master" {
  ami           = "ami-0c2b8ca1dad447f8a" # Ubuntu 22.04 in us-east-1
  instance_type = "t2.medium"
  key_name      = var.key_pair_name
  tags = {
    Name = "jenkins-master"
  }
}
```

#### Commands:
```bash
terraform init
terraform apply
```

---

### 2. ☕ Install Java

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

---

### 3. ⚙️ Install Jenkins

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
```

Add Jenkins to Docker and sudo:
```bash
sudo usermod -aG docker jenkins
sudo usermod -aG sudo jenkins
```

---

### 4. 🌐 Install and Configure NGINX Reverse Proxy

```bash
sudo apt install nginx -y
sudo systemctl start nginx
```

Paste the reverse proxy config in: `/etc/nginx/conf.d/jenkins.conf`

🧪 Test and reload:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

### 5. 🔐 Set Up UFW Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

---

### 6. 🔒 Install Certbot for SSL

```bash
sudo snap install core; sudo snap refresh core
sudo apt remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx -d jenkins.lilianedevops.online
```

🔁 Auto-renew test:
```bash
sudo certbot renew --dry-run
```

---

### 7. 🧩 Jenkins Agent Setup (EC2)

#### a. Provision another EC2 instance for agent

#### b. On both master and agent:
```bash
sudo adduser jenkins
sudo usermod -aG sudo jenkins
```

#### c. On master (Jenkins user):
```bash
sudo -u jenkins ssh-keygen -t rsa
```

#### d. On agent:
Paste master’s public key into:
```bash
~/.ssh/authorized_keys
```

#### e. Permissions:
```bash
sudo chown -R jenkins /var/lib/jenkins/.ssh
sudo chmod 600 /var/lib/jenkins/.ssh/authorized_keys
sudo chmod 700 /var/lib/jenkins/.ssh
```

#### f. Test SSH:
```bash
sudo -u jenkins ssh jenkins@<agent-ip>
```

#### g. Jenkins UI:
- Go to **Manage Jenkins → Nodes**
- Create agent with label: `linux-agent`
- Use SSH credentials (private key from master)

---

### 8. ✅ Jenkins Deep Dive + Run Demo Job on Agent

#### a. Install Plugins:
- Docker Pipeline
- Blue Ocean
- Git, GitHub Integration
- SSH Agent

#### b. Restart Jenkins:
```bash
sudo systemctl restart jenkins
```

#### c. Create Pipeline Job with the following `Jenkinsfile`:

```groovy
pipeline {
  agent { label 'linux-agent' }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/lily4499/demo-node-app.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          docker.build('demo-node-app')
        }
      }
    }
    stage('Run Container') {
      steps {
        sh 'docker run -d -p 3000:3000 demo-node-app'
      }
    }
  }
}
```

#### d. Access your app:

```bash
http://<agent-ip>:3000
```

Or if reverse proxied:

```bash
https://jenkins.lilianedevops.online
```

---

## 🔁 Recap

| Task                     | Description                                      |
|--------------------------|--------------------------------------------------|
| Terraform Infra          | Automates provisioning of master & agent EC2s   |
| Jenkins Setup            | Installs Jenkins with required plugins           |
| NGINX + Certbot          | Secures Jenkins UI with HTTPS                   |
| Jenkins Agent            | Enables distributed builds                       |
| Pipeline Demo            | Runs Dockerized job on Jenkins agent            |

---


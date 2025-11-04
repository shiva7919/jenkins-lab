

# 🚀 Jenkins Multi-Server CI/CD Infrastructure Setup

### *(Java 17 + Maven 3.9.11 + SonarQube + Nexus + Tomcat)*

---

## 🧭 Overview

This documentation describes how to set up a **multi-server CI/CD infrastructure** using **Jenkins** as the central automation server.  
It automates the complete software lifecycle — from code build, quality analysis, artifact storage, and deployment — through the following workflow:

```

GitHub → Jenkins → SonarQube → Nexus → Tomcat

```

---

## 🏗️ Architecture Diagram

```
           +----------------+
           |   Developer    |
           |  (GitHub Push) |
           +-------+--------+
                   |
                   v
           +----------------+
           |   Jenkins CI   |  (Main Server)
           |  Java + Maven  |
           +----------------+
            /      |       \
           /       |        \
SSH to -> /        |         \ -> SSH to

+---------------+  +---------------+  +---------------+
|   SonarQube   |  |    Nexus      |  |    Tomcat     |
| Code Quality  |  | Artifact Repo |  | App Deployment|
+---------------+  +---------------+  +---------------+
```
---

## ⚙️ Server Setup

| Server Name | Purpose | Hostname |
|--------------|----------|-----------|
| Jenkins | CI/CD Orchestrator | `jenkins` |
| SonarQube | Code Quality Analysis | `sonar` |
| Nexus | Artifact Repository | `nexus` |
| Tomcat | Application Deployment | `tomcat` |

---

## 🖥️ Step 1: Configure Hostnames

Run these commands on **each EC2 instance**:

```bash
sudo hostnamectl set-hostname jenkins
sudo hostnamectl set-hostname sonar
sudo hostnamectl set-hostname nexus
sudo hostnamectl set-hostname tomcat
sudo init 6
````

### ✅ Hostname Setup Screenshots

**Jenkins:**

<img width="951" height="133" alt="Jenkins Hostname" src="https://github.com/user-attachments/assets/2edf80e8-bde4-4cdb-a939-7a5a59b24416" />

**Sonar:**

<img width="959" height="190" alt="Sonar Hostname" src="https://github.com/user-attachments/assets/61bc1a73-66f9-4d97-b35e-59b73a6eb65a" />

**Nexus:**

<img width="987" height="150" alt="Nexus Hostname" src="https://github.com/user-attachments/assets/bd8c26bb-d9d2-45c8-a659-6aed6a6fa6d1" />

**Tomcat:**

<img width="1219" height="210" alt="Tomcat Hostname" src="https://github.com/user-attachments/assets/b4980e88-5898-4091-8c8a-cd7c0b49f9ac" />

---

## ☕ Step 2: Install Java 17 and Maven 3.9.11

Run on **all servers**:

```bash
sudo apt update -y
sudo apt install openjdk-17-jdk -y
java -version
sudo apt install maven -y
mvn -version
```

✅ Verify both Java and Maven versions are displayed correctly.

---

## 📁 Step 3: Create Jenkins Directory

```bash
mkdir jenkins
```

---

## 🔑 Step 4: Configure SSH Key-Based Authentication

Generate SSH key on **Jenkins server**:

```bash
ssh-keygen
```

Copy the public key to other servers:

```bash
ssh-copy-id ubuntu@<Sonar_Server_IP>
ssh-copy-id ubuntu@<Nexus_Server_IP>
ssh-copy-id ubuntu@<Tomcat_Server_IP>
```

### ✅ SSH Key Generation & Copy Screenshots

**Key generated on Jenkins:**

<img width="1304" height="165" alt="SSH Key Generated" src="https://github.com/user-attachments/assets/c79fc7bb-04e1-40da-977b-a7c988e906cf" />
<img width="1576" height="635" alt="SSH Key Generation Output" src="https://github.com/user-attachments/assets/959d8ad5-2f2f-46bc-8705-a93fffbe1518" />

**Keys copied to remote servers:**

* **Sonar:** <img width="1904" height="1018" alt="Sonar SSH Key Copy" src="https://github.com/user-attachments/assets/73d40fb8-845f-4c37-8d8a-baf4abd48246" />

* **Nexus:** <img width="1910" height="344" alt="Nexus SSH Key Copy" src="https://github.com/user-attachments/assets/fb11fbf9-687f-4667-8660-9669f0718a76" />

* **Tomcat:** <img width="1907" height="402" alt="Tomcat SSH Key Copy" src="https://github.com/user-attachments/assets/872c855e-bc72-4f74-bfc1-54a05cd83d03" />

---

## 🧩 Step 5: Jenkins Setup and Configuration

Access Jenkins at:

```
http://<Jenkins_Server_IP>:8080
```

### Unlock Jenkins

<img width="1911" height="994" alt="Unlock Jenkins" src="https://github.com/user-attachments/assets/5e9298ee-0434-4b25-801e-cff65da4f209" />

To view Jenkins initial password:

<img width="1149" height="146" alt="Jenkins Password" src="https://github.com/user-attachments/assets/26c1b4a7-fb30-4290-a521-8102d79287e9" />

### Customize Jenkins

<img width="1920" height="1006" alt="Customize Jenkins" src="https://github.com/user-attachments/assets/56d23ced-1ec5-40a9-b08f-5c8e64b21249" />

### Getting Started

<img width="1920" height="1007" alt="Jenkins Getting Started" src="https://github.com/user-attachments/assets/2c5f0678-e23e-4aea-9821-7097b09dfaee" />

### Create First Admin User

<img width="1920" height="1009" alt="Create First Admin User" src="https://github.com/user-attachments/assets/504e794b-658d-4ca1-a6fe-abdda7bfc97c" />

### Jenkins Status

<img width="1920" height="744" alt="Jenkins Status" src="https://github.com/user-attachments/assets/50f7e206-e13c-4e6e-9790-3e4df41111ec" />

---

## 🔌 Step 6: Install Jenkins Plugins

Navigate to:

```
Manage Jenkins → Plugins → Available Plugins
```

### Recommended Plugins

| Plugin                  | Purpose              |
| ----------------------- | -------------------- |
| Publish Over SSH        | Artifact deployment  |
| GitHub Integration      | SCM connection       |
| SonarQube Scanner       | Code analysis        |
| Nexus Artifact Uploader | Upload to Nexus      |
| SSH Build Agents        | Connect remote nodes |

### Installed Plugins Screenshot

<img width="1118" height="539" alt="Screenshot 2025-11-04 151813" src="https://github.com/user-attachments/assets/cfb9233c-905b-44a3-83b1-696fb1ae778f" />


---

## 🧠 Step 7: Configure Jenkins Nodes

Path:

```
Manage Jenkins → Nodes → New Node
```

| Node   | Host          | Label  | Directory              |
| ------ | ------------- | ------ | ---------------------- |
| Sonar  | `<Sonar_IP>`  | sonar  | `/home/ubuntu/jenkins` |
| Nexus  | `<Nexus_IP>`  | nexus  | `/home/ubuntu/jenkins` |
| Tomcat | `<Tomcat_IP>` | tomcat | `/home/ubuntu/jenkins` |

**Launch Method:** via SSH using `jenkins_ssh_key`.

### Nodes Configuration Output

<img width="1893" height="883" alt="Jenkins Nodes" src="https://github.com/user-attachments/assets/65578b70-f6ca-42e3-8540-fdcaf8b8832c" />

---
## step 9 GitHub Webhook Setup

1. In Jenkins → **New Item → Pipeline**
<img width="1920" height="1080" alt="59" src="https://github.com/user-attachments/assets/802ca99d-b61a-447a-b3bb-2dd485fe64ed" />

   * Definition: *Pipeline script from SCM*
   * SCM: *Git*
   * Repository URL: `https://github.com/you/your-app.git`
   * Branch: `*/main`
   * Script path: `Jenkinsfile`
2. Check **GitHub hook trigger for GITScm polling**.
3. In your GitHub repo → **Settings → Webhooks → Add webhook**

   * Payload URL: `http://<JENKINS_PUBLIC_IP>:8080/github-webhook/`
   * Content type: `application/json`
   * Event: *Push events*
4. Push a commit — the pipeline runs automatically.

<img width="1920" height="1080" alt="60" src="https://github.com/user-attachments/assets/09b79dde-affc-44b9-93be-9b8fb7e7e067" />
 

## 🧩 Step 9: Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent none

    stages {
        stage('Sonar Stage') {
            agent { label 'sonar' }
            steps {
                script {
                    echo "=== Running on SonarQube Agent ==="
                    sh '''
                        echo "Current User:"; whoami
                        echo "Host IP:"; hostname -i
                    '''
                }
            }
        }

        stage('Nexus Stage') {
            agent { label 'nexus' }
            steps {
                script {
                    echo "=== Running on Nexus Agent ==="
                    sh '''
                        echo "Current User:"; whoami
                        echo "Host IP:"; hostname -i
                    '''
                }
            }
        }

        stage('Tomcat Stage') {
            agent { label 'tomcat' }
            steps {
                script {
                    echo "=== Running on Tomcat Agent ==="
                    sh '''
                        echo "Current User:"; whoami
                        echo "Host IP:"; hostname -i
                    '''
                }
            }
        }
    }
}
```

---

## 🧾 Log Outputs

**Sonar Log:**

<img width="1911" height="1001" alt="Sonar Log" src="https://github.com/user-attachments/assets/93b6aef1-a434-4518-865c-6ad1c60414ed" />

**Nexus Log:**

<img width="1909" height="1014" alt="Nexus Log" src="https://github.com/user-attachments/assets/2f43f961-b00f-4896-b5f0-50edbcf7142c" />

**Tomcat Log:**

<img width="1883" height="1018" alt="Tomcat Log" src="https://github.com/user-attachments/assets/6f6032f6-3bfa-414b-ad91-043d18b663c6" />

---

## 🧾 Pipeline Console Outputs

**Sonar Stage Output:**

<img width="1903" height="1014" alt="Sonar Pipeline Output" src="https://github.com/user-attachments/assets/cf3498c8-f138-4b16-be48-8708a7646f0e" />

**Nexus Stage Output:**

<img width="1920" height="984" alt="Nexus Pipeline Output" src="https://github.com/user-attachments/assets/181e6ce7-e319-476f-8c74-5b859e0085b0" />

**Tomcat Stage Output:**

<img width="1904" height="1000" alt="Tomcat Pipeline Output" src="https://github.com/user-attachments/assets/6684c56a-ffcf-4fe7-9b77-c172859222c0" />

---

## 🏁 Final Results

✅ 4 servers fully connected via SSH
✅ Jenkins orchestrating Sonar, Nexus, and Tomcat
✅ Verified Java, Maven, and agent connectivity
✅ Ready for GitHub → Jenkins → SonarQube → Nexus → Tomcat CI/CD flow

---

## 🌐 Next Steps (Enhancements)

| Enhancement                             | Benefit               |
| --------------------------------------- | --------------------- |
| Integrate SonarQube + Nexus in pipeline | Full CI/CD automation |
| Add Email/Slack notifications           | Real-time alerts      |
| Configure RBAC                          | Secure Jenkins access |
| Add backups                             | Recovery-ready setup  |
| Integrate Prometheus + Grafana          | Monitoring            |

---

## 📘 Summary

This setup delivers a **modular, scalable, and secure CI/CD environment** powered by Jenkins.

> 🧠 Jenkins handles automation → SonarQube ensures code quality → Nexus stores artifacts → Tomcat deploys apps.

---

## 💡 Author

**Shiva Sai Kumar**
DevOps Engineer | CI/CD | Cloud Automation

---

#  CI/CD Pipeline: GitHub → Jenkins → SonarQube → Nexus → Tomcat  

![Jenkins](https://img.shields.io/badge/Jenkins-Automation-blue?logo=jenkins)
![SonarQube](https://img.shields.io/badge/SonarQube-Quality%20Analysis-success?logo=sonarqube)
![Nexus](https://img.shields.io/badge/Nexus-Repository-orange?logo=sonatype)
![Tomcat](https://img.shields.io/badge/Tomcat-Deployment-yellow?logo=apachetomcat)
![Maven](https://img.shields.io/badge/Maven-Build-red?logo=apachemaven)
![Java](https://img.shields.io/badge/Java-17-brightgreen?logo=java)

---
# 🔎 Deep Dive: How This Pipeline Works (and How to Keep It Healthy)

## 1️⃣ Why This Design?

- **Clear separation of concerns:**  
  Each server handles a specific responsibility —  
  Build & analysis (**Sonar node**), storage (**Nexus**), and runtime (**Tomcat**).

- **Traceability & repeatability:**  
  Every build is versioned as `0.0.${BUILD_NUMBER}` and stored in **Nexus**, ensuring reproducible deployments.

- **Fast feedback:**  
  **SonarQube Quality Gate** ensures code quality is checked early — stopping bad code before deployment.

- **Idempotent deploy:**  
  The **Tomcat undeploy → deploy** flow ensures clean, predictable updates without leftover artifacts.

---

## 2️⃣ Required Versions & Ports (Quick Checklist)

| **Component** | **Recommended Version** | **Ports** |
|----------------|--------------------------|------------|
| Jenkins LTS | Latest LTS | `8080` (HTTP), `50000` (agents JNLP, if used) |
| Java (JDK) | 17 (match your tools) | — |
| Maven | 3.9.x | — |
| SonarQube | 9.x+ (Community OK) | `9000` |
| Nexus OSS 3 | Latest 3.x | `8081` |
| Tomcat | 9 / 10 | `8080` |

---

## 🌐 Network Rules (Open Between Servers)

| **From → To** | **Port** | **Purpose** |
|----------------|-----------|--------------|
| Jenkins → SonarQube | `9000` | Code analysis communication |
| Jenkins (agents) ↔ Jenkins Master | `22 (SSH)` or `50000 (JNLP)` | Node connection |
| Jenkins → Nexus | `8081` | Artifact upload |
| Jenkins/Tomcat Node → Nexus | `8081` | Artifact download (WAR fetch) |
| Jenkins → Tomcat Manager | `8080` | Application deployment |
| GitHub → Jenkins | `8080/github-webhook/` | Webhook trigger for pipeline |

---

💡 **Tip:**  
Ensure all servers can resolve each other’s **private IPs or hostnames**.  
Use `ping`, `curl`, or `telnet` to test connectivity between Jenkins, SonarQube, Nexus, and Tomcat nodes.

---


## 🎯 **Objective**

This project demonstrates a **complete CI/CD pipeline** using Jenkins that automatically integrates:
- **GitHub** → code repository & webhook trigger  
- **SonarQube** → static code quality analysis  
- **Maven** → build automation and artifact packaging  
- **Nexus** → artifact repository management  
- **Tomcat** → deployment of the final WAR file  

✅ **Everything is automated** — from code push to deployment on Tomcat.

---

## 🧩 **Workflow Overview**

```text
GitHub (Push) 
     ↓
Jenkins Master
├── Sonar Node (Build + Analysis)
│     ↳ SonarQube Scan
│     ↳ Maven Build (WAR)
│     ↳ Upload to Nexus
│└── Nexus (Binary Repository)
│      ↳ Artifact is uploaded
│      ↳ Artifacr is deployed to Tomcat 
└── Tomcat Node (Deployment)
       ↳ Pull latest WAR from Nexus
       ↳ Deploy on Apache Tomcat

````

---

## ⚙️ **Technology Stack**

| Component | Purpose                 |
| --------- | ----------------------- | 
| Jenkins   | CI/CD Automation Server |
| SonarQube | Code Quality Analysis   | 
| Nexus OSS | Artifact Repository     | 
| Tomcat    | Web Application Server  |
| Maven     | Build Tool              |
| Java      | Runtime                 | 

---

## 🧱 **Infrastructure Setup**

| Server         | Purpose             |
| -------------- | ------------------- | 
| Jenkins Master | CI/CD Orchestrator  | 
| Sonar Node     | Build + Analysis    | 
| Nexus Server   | Artifact Repository | 
| Tomcat Server  | Deployment Target   |

---

## 🧰 **Jenkins Configuration Summary**

### Plugins Required:

* Git
* GitHub Integration
* Pipeline
* Maven Integration
* SonarQube Scanner
* SSH Agents
* Credentials Binding

### Global Tools Configuration:

| Tool  | Name  | Notes                  |
| ----- | ----- | ---------------------- |
| JDK   | JDK17 | Installed on all nodes |
| Maven | Maven | Auto-install or local  |

### Credentials:

| ID             | Type              | Description            |
| -------------- | ----------------- | ---------------------- |
| `nexus`        | Username/Password | Nexus artifact repo    |
| `tomcat`       | Username/Password | Tomcat manager         |
| `ssh-ubuntu`   | SSH key           | Jenkins → Agents       |
| `github-token` | Secret text       | GitHub webhook trigger |

---
 ## 📁 **Jenkinsfile (Pipeline Script)**

```groovy
pipeline {
    agent { label 'sonar' }

    tools {
        jdk 'JDK17'
        maven 'Maven'
    }

    environment {
        SONARQUBE_SERVER = 'sonar'
        MVN_SETTINGS = '/etc/maven/settings.xml'
        NEXUS_URL = 'http://54.234.225.51:8081'
        NEXUS_REPO = 'maven-releases'
        NEXUS_GROUP = 'com/web/cal'
        NEXUS_ARTIFACT = 'webapp-add'
        TOMCAT_URL = 'http://34.207.110.47:8080/manager/text'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📦 Cloning source from GitHub...'
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/shiva7919/Java-Web-Calculator-App.git']]
                ])
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Running SonarQube static analysis...'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn clean verify sonar:sonar -DskipTests --settings ${MVN_SETTINGS}'
                }
            }
        }

        stage('Build Artifact') {
            steps {
                echo '⚙️ Building WAR...'
                sh 'mvn clean package -DskipTests --settings ${MVN_SETTINGS}'
                sh 'echo ✅ Build Completed!'
                sh 'ls -lh target/*.war || true'
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USR', passwordVariable: 'NEXUS_PSW')]) {
                    sh '''#!/bin/bash
                        set -e
                        WAR_FILE=$(ls target/*.war | head -1)
                        FILE_NAME=$(basename "$WAR_FILE")
                        VERSION="0.0.${BUILD_NUMBER}"

                        echo "📤 Uploading $FILE_NAME to Nexus as version $VERSION..."

                        curl -v -u ${NEXUS_USR}:${NEXUS_PSW} --upload-file "$WAR_FILE" \
                          "${NEXUS_URL}/repository/${NEXUS_REPO}/${NEXUS_GROUP}/${NEXUS_ARTIFACT}/${VERSION}/${NEXUS_ARTIFACT}-${VERSION}.war"

                        echo "✅ Artifact uploaded successfully to Nexus!"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            agent { label 'tomcat' }
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USR', passwordVariable: 'NEXUS_PSW'),
                    usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USR', passwordVariable: 'TOMCAT_PSW')
                ]) {
                    sh '''#!/bin/bash
                        set -e
                        cd /tmp; rm -f *.war

                        echo "🔍 Fetching latest WAR from Nexus..."
                        DOWNLOAD_URL=$(curl -s -u ${NEXUS_USR}:${NEXUS_PSW} \
                            "${NEXUS_URL}/service/rest/v1/search?repository=${NEXUS_REPO}&group=com.web.cal&name=webapp-add" \
                            | grep -oP '"downloadUrl"\\s*:\\s*"\\K[^"]+\\.war' | grep -vE '\\.md5|\\.sha1' | tail -1)

                        if [[ -z "$DOWNLOAD_URL" ]]; then
                            echo "❌ No WAR found in Nexus!"
                            exit 1
                        fi

                        echo "⬇️ Downloading WAR: $DOWNLOAD_URL"
                        curl -u ${NEXUS_USR}:${NEXUS_PSW} -O "$DOWNLOAD_URL"
                        WAR_FILE=$(basename "$DOWNLOAD_URL")
                        APP_NAME=$(echo "$WAR_FILE" | sed 's/-[0-9].*//')

                        echo "🧹 Removing old deployment..."
                        curl -u ${TOMCAT_USR}:${TOMCAT_PSW} "${TOMCAT_URL}/undeploy?path=/${APP_NAME}" || true

                        echo "🚀 Deploying new WAR to Tomcat..."
                        curl -u ${TOMCAT_USR}:${TOMCAT_PSW} --upload-file "$WAR_FILE" \
                            "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"

                        echo "✅ Deployment successful! Application updated."
                    '''
                }
            }
        }
    }

    post {
        success { echo '🎉 Pipeline completed successfully — Application live on Tomcat!' }
        failure { echo '❌ Pipeline failed — Check Jenkins logs.' }
    }
}
```

---

## 🧾 **Validation Checklist**

* ✅ Jenkinsfile present in repository root
* ✅ GitHub webhook configured at: `http://<JENKINS_IP>:8080/github-webhook/`
* ✅ SonarQube accessible at: `http://<SONAR_IP>:9000`
* ✅ Nexus running at: `http://54.234.225.51:8081`
* ✅ Tomcat accessible at: `http://34.207.110.47:8080/manager/html`
* ✅ Jenkins agents online: `sonar`, `tomcat`

---

## 🩵 **Sample Pipeline Output**

```
📦 Cloning source from GitHub...
🔍 Running SonarQube static analysis...
⚙️ Building WAR...
📤 Uploading artifact to Nexus...
⬇️ Downloading WAR from Nexus...
🚀 Deploying new WAR to Tomcat...
🎉 Pipeline completed successfully — Application live on Tomcat!
```
## Screenshots
Githubwebhook integration:
<img width="1919" height="1079" alt="Screenshot 2025-11-05 150542" src="https://github.com/user-attachments/assets/87e7dc1a-71af-4780-a049-f7d4a827f035" />

Build Pipeline in Jenkins
<img width="1919" height="1079" alt="Screenshot 2025-11-05 145823" src="https://github.com/user-attachments/assets/bd797824-c7d3-4064-92dd-8631efc2e8cc" />

<img width="1919" height="1079" alt="Screenshot 2025-11-05 150030" src="https://github.com/user-attachments/assets/113a1280-8b62-4b05-ba6e-c59cda3f03fc" />

Sonarqube test pass:
<img width="1919" height="1079" alt="Screenshot 2025-11-05 150220" src="https://github.com/user-attachments/assets/e3b7acb1-c6ea-4185-999f-bdc85b0e751d" />

Nexus artifact stored:
<img width="1919" height="1079" alt="Screenshot 2025-11-05 150320" src="https://github.com/user-attachments/assets/9f7fe4e9-4a4a-4282-91a3-785c6ac0d544" />

Tomcat before the deploy:
<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/c9956e47-6fb9-4d72-a9f6-0dd362a8f33e" />

Tomcat after deploy:
<img width="1920" height="1080" alt="5" src="https://github.com/user-attachments/assets/fdf8cb0b-3984-448e-9e89-a2842515321b" />

Final deployment:
<img width="1920" height="1080" alt="6" src="https://github.com/user-attachments/assets/18042574-ac10-4b27-9ad1-9c0c7d208203" />

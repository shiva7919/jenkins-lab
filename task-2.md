# CI/CD Pipeline Using Jenkins and GitHub Webhooks

## Overview

This project demonstrates a complete CI/CD pipeline built using Jenkins. The pipeline automates the process of cloning source code, testing it, building it into deployable artifacts, and finally deploying it to the target environment.

## Source Code Repository :https://github.com/shiva7919/Java-Web-Calculator-App.git

This pipeline uses the following GitHub repository as the source:


**Repository:** 
<img width="1891" height="1055" alt="8" src="https://github.com/user-attachments/assets/d8c1d545-214e-4b95-b058-cd0e03f224bc" />
## Pipeline Stages

1. **Cloning Job**

   * Fetches the latest code from GitHub.
<img width="1920" height="1080" alt="16" src="https://github.com/user-attachments/assets/6c2b42c9-aac1-456e-9f24-53ee714a0594" />
<img width="1920" height="1080" alt="17" src="https://github.com/user-attachments/assets/0a8a68d3-157b-4352-a620-c6097e37cb06" />
<img width="1920" height="1080" alt="18" src="https://github.com/user-attachments/assets/8afbcaae-dd67-484d-8a71-60842f611121" />
<img width="1919" height="1079" alt="15 clone" src="https://github.com/user-attachments/assets/2b72cd68-9632-4c8d-85ad-c64899d1de03" />

2. **Test Job**

   * Executes automated tests to verify the integrity and functionality of the code.
<img width="1920" height="1080" alt="23" src="https://github.com/user-attachments/assets/b0a20702-cab5-42e4-9038-84b960ff581b" />
<img width="1920" height="1080" alt="24" src="https://github.com/user-attachments/assets/5924dc5d-e461-40b5-b569-4422f2d12b0a" />
<img width="1920" height="1080" alt="25" src="https://github.com/user-attachments/assets/ea6af38d-ffdc-4318-84f7-e16c785f47e3" />
 <img width="1919" height="1079" alt="17 test" src="https://github.com/user-attachments/assets/cbd90da5-3cf8-404b-bc8e-6c77d4c23f04" />


3. **Build Job**

   * Builds the project and creates the output artifacts.
<img width="1920" height="1080" alt="27" src="https://github.com/user-attachments/assets/10ff86c5-fa5c-45b9-b302-055f67ec283d" />
<img width="1919" height="1079" alt="19 build" src="https://github.com/user-attachments/assets/917c5507-9a2a-4fcf-aac7-f52db038140b" />


4. **Deploy Job**

   * Deploys the final build to the target server/environment.
<img width="1920" height="1080" alt="29" src="https://github.com/user-attachments/assets/3a29c155-9b57-4d1f-bb3f-9576172bbca4" />
<img width="1919" height="1079" alt="22 deploy" src="https://github.com/user-attachments/assets/53b75c1c-d947-42f4-8bb1-ad8e059c640f" />


## Server Details

The Jenkins server was hosted on an **AWS EC2 instance (t2.micro)**.
<img width="1914" height="1079" alt="1" src="https://github.com/user-attachments/assets/cba3b0e5-5b47-48c6-b123-4132a642f34a" />


## Jenkins Installation Script

The following script was used to install Jenkins on an Ubuntu server:
<img width="1112" height="446" alt="2" src="https://github.com/user-attachments/assets/e6eca73b-6dc2-4d0d-8b57-7ef572bc7fb7" />

```bash
#!/bin/bash

sudo apt update
sudo apt install fontconfig openjdk-21-jre -y

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

echo "---------------------------Jenkins has been installed ------------------------"
```
<img width="1087" height="330" alt="3" src="https://github.com/user-attachments/assets/abb06c85-c237-4348-914b-a24a7c96ae95" />


## Jenkins Setup Steps

1. Installed Jenkins using a `script.sh` file.
2. Accessed Jenkins via `http://<server-ip>:8080`.
3. Retrieved Jenkins initial admin password from the system path.
4. Created admin credentials for Jenkins dashboard usage.
5. Created the four jobs listed above.
6. Installed the following plugins:

   * Build Pipeline Plugin
   * GitHub Plugin
   * GitHub Webhook Plugin

<img width="1920" height="1080" alt="36" src="https://github.com/user-attachments/assets/ac8ec640-ef3e-43d3-ad86-2e706def0736" />

<img width="1920" height="1080" alt="39" src="https://github.com/user-attachments/assets/87f8bea8-a1bf-41f1-8978-e7992c585ad5" />

7. Configured pipeline view to visualize and trigger jobs sequentially.

## GitHub Webhook Integration

* Configured a webhook in GitHub to notify Jenkins upon every code push.
* Jenkins automatically triggered the pipeline when changes were pushed.

<img width="1919" height="1079" alt="24" src="https://github.com/user-attachments/assets/bbfa451a-44e8-4451-99e4-c2f27313ace1" />
<img width="994" height="239" alt="25" src="https://github.com/user-attachments/assets/122a2d50-d814-4860-8382-80f235781a0c" />

## Pipeline Flow

```
GitHub Commit → Clone Job → Test Job → Build Job → Deploy Job
```
<img width="1920" height="1080" alt="41" src="https://github.com/user-attachments/assets/6b5a77dd-b6f5-4b56-a1e1-83c9e24c6733" />
<img width="1920" height="1080" alt="42" src="https://github.com/user-attachments/assets/898355af-71d7-4931-ae98-6459735778d2" />
<img width="1920" height="1080" alt="47" src="https://github.com/user-attachments/assets/62bd03d2-be70-4058-a084-91146d20574c" />
<img width="1920" height="1080" alt="48" src="https://github.com/user-attachments/assets/caac05f2-e727-4850-8b59-fd8e118e4b49" />
<img width="1920" height="1080" alt="49" src="https://github.com/user-attachments/assets/23a24d5f-953a-4bfb-903a-7c780c62f3d3" />

## Conclusion

This CI/CD setup ensures automated, repeatable, and reliable software delivery triggered directly from source code changes. It helps maintain consistency, reduces manual interventions, and increases deployment efficiency.

---

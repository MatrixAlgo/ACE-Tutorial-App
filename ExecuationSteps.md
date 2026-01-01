
# CI/CD Automation for IBM App Connect Enterprise (ACE)
## Execution Guide

---

## Purpose

Automatically build and deploy IBM App Connect Enterprise (ACE) applications when a developer pushes code to GitHub.

---

## CI/CD Flow

```
Developer → GitHub → Webhook → ngrok → Jenkins → BAR Build → ACE Deploy
```

---

## Prerequisites

### 1. Operating System
- Ubuntu Linux (18.04+ recommended)

### 2. Software Installed
- Git
- Java (required by Jenkins)
- Jenkins (running on port 8080)
- IBM App Connect Enterprise (ACE) 12.x
- ngrok (latest version)

### 3. Accounts
- GitHub account
- ngrok account (free tier is sufficient)

### 4. Network
- Jenkins running locally
- Outbound internet access from Jenkins host

---

## Step 1: Install & Verify IBM ACE

**Example Installation Path**
```bash
/opt/IBM/ace-12.0.12.19
```

**Verify ACE**
```bash
source /opt/IBM/ace-12.0.12.19/server/bin/mqsiprofile
ibmint version
mqsilist
```

**Expected Output**
- Integration node listed (example: `ACE_NODE`)

---

## Step 2: Start ACE Node & Integration Server

### Start Integration Node
```bash
mqsistart ACE_NODE
```

### Create Integration Server (if not exists)
```bash
mqsicreateexecutiongroup ACE_NODE -e IS01
```

### Verify
```bash
mqsilist ACE_NODE
```

---

## Step 3: Install & Start Jenkins

### Start Jenkins
```bash
sudo systemctl start jenkins
```

### Access Jenkins UI
```
http://localhost:8080
```

### Install Required Jenkins Plugins
- Git Plugin
- Pipeline
- Generic Webhook Trigger

---

## Step 4: Create Jenkins Pipeline Job

1. Jenkins Dashboard → **New Item**
2. Select **Pipeline**
3. Job Name: `ACE-CICD`
4. Select **Pipeline script from SCM**
5. SCM: **Git**
6. Repository URL:
   ```
   https://github.com/<ORG>/<REPO>.git
   ```
7. Credentials: GitHub Personal Access Token (PAT)
8. Branch: `main`
9. Script Path:
   ```
   Jenkinsfile
   ```

---

## Step 5: Jenkinsfile (Pipeline Script)

Create a file named **`Jenkinsfile`** in the Git repository root.

```groovy
pipeline {
    agent any

    environment {
        ACE_NODE   = "ACE_NODE"
        ACE_SERVER = "IS01"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build BAR') {
            steps {
                sh '''
                bash -lc '
                set -e
                source /opt/IBM/ace-12.0.12.19/server/bin/mqsiprofile

                mkdir -p build
                BAR_NAME=$(basename $(pwd))_${BUILD_NUMBER}.bar

                ibmint package                   --input-path $(pwd)                   --output-bar-file build/$BAR_NAME
                '
                '''
            }
        }

        stage('Deploy BAR') {
            steps {
                sh '''
                bash -lc '
                set -e
                source /opt/IBM/ace-12.0.12.19/server/bin/mqsiprofile

                BAR_FILE=$(ls -t build/*.bar | head -1)
                mqsideploy $ACE_NODE -e $ACE_SERVER -a $BAR_FILE -w 120
                '
                '''
            }
        }
    }
}
```

---

## Step 6: Configure Generic Webhook Trigger in Jenkins

Jenkins Job → **Configure** → **Build Triggers**

Enable:
```
[x] Generic Webhook Trigger
```

Token:
```
ace-pipeline-123
```

> Leave all other fields empty.

---

## Step 7: Install & Configure ngrok

### Download ngrok
```
https://ngrok.com/download
```

### Configure Auth Token
```bash
ngrok authtoken <YOUR_AUTH_TOKEN>
```

### Expose Jenkins Port
```bash
ngrok http 8080
```

**Example Output**
```
https://abcd-1234.ngrok-free.dev
```

---

## Step 8: Configure GitHub Webhook

GitHub Repository → **Settings** → **Webhooks** → **Add webhook**

### Payload URL
```
https://abcd-1234.ngrok-free.dev/generic-webhook-trigger/invoke?token=ace-pipeline-123
```

### Content Type
```
application/json
```

### Events
- [x] Push events

### SSL
- Enable SSL verification

Click **Save webhook**

---

## Step 9: Test Webhook Manually

### Local Test
```bash
curl -X POST http://localhost:8080/generic-webhook-trigger/invoke?token=ace-pipeline-123
```

### Expected Response
```
Triggered jobs.
```

---

## Step 10: End-to-End Test

1. Modify any file (example: `README.md`)
2. Commit and push changes:
```bash
git add .
git commit -m "Test CI/CD"
git push origin main
```

### Expected Results
- GitHub webhook delivery **SUCCESS (200/202)**
- Jenkins pipeline starts automatically
- BAR file created in `build/`
- BAR deployed to ACE integration server `IS01`
- Application running successfully in ACE

---

## Troubleshooting

### Webhook 404 Errors
- Jenkins job trigger not enabled
- Incorrect ngrok URL
- Jenkins not running

### `mqsideploy` Errors
- Multiple BAR files present
- ACE integration node not started
- Incorrect integration server name

### Logs to Check
- Jenkins **Console Output**
- ACE logs:
```bash
/var/mqsi/common/log
```

---

## Final Summary

```
Developer Push
→ GitHub Webhook
→ ngrok Tunnel
→ Jenkins Pipeline
→ BAR Build
→ Deploy to IBM ACE
→ Application Live Automatically
```

---

## End of Document

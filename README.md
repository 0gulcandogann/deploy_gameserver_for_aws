# ☁️ Deploying Minecraft Server to AWS GameLift via Ansible Automation Platform

This repository contains a playbook designed to deploy a **Vanilla Minecraft server to AWS GameLift** using **Ansible Automation Platform (AAP / AWX)**.

The primary goal is to fully automate all AWS-side operations (IAM, S3, GameLift Fleet, Autoscaling).

This guide covers two parts:
1.  **Manual Setup:** How to configure AAP/AWX to run this playbook.
2.  **Automated Deployment:** How to set up a GitHub Actions workflow to automatically run the playbook on a `git push`.

---

## 🎯 What It Does

This playbook essentially does the following:

- Creates the necessary **IAM roles and policies** for GameLift on AWS.
- Creates an **S3 bucket** and uploads the Minecraft server build to it.
- Adds the **build** record to GameLift and creates a **Fleet**.
- Finally, it configures the **Autoscaling** policies.

So, we are turning "a job you would do with 20 commands locally" into a single Job Template in AAP.

---

## ⚙️ Requirements

To run this setup, you need the following:

- AWS account (GameLift must be enabled)
- IAM user/role (required permissions: `gamelift:*`, `iam:*`, `ec2:*`, `s3:*`, `autoscaling:*`)
- AAP (Ansible Automation Platform) or an open-source AWX installation
- A credential tested with AWS CLI (we will add this to AAP)

---

## 🧠 Part 1: Manual AAP/AWX Setup

Before you can automate the *trigger*, you must first set up the project and template in AAP/AWX.

### 1️⃣ Import the Repository Content

Create a **Project** in AWX / AAP:
- **Type:** Git
- **SCM URL:** The URL of this repository
- **Branch:** main (or your branch)

### 2️⃣ Add AWS Credential

AAP needs credentials to talk to AWS.

In the AAP Dashboard:
**Credentials → Add → Amazon Web Services**
- **Name:** `My AWS Credentials` (or similar)
- **Access Key ID:** (Your AWS Key)
- **Secret Access Key:** (Your AWS Secret)
- **Default region:** (e.g., `eu-central-1`)

**Note:** These AWS credentials stay *inside* AAP. They are never exposed to GitHub.

### 3️⃣ Create Job Template

This template connects the Project (playbook), Inventory (where to run), and Credentials (how to authenticate).

Add a new Job Template in AAP:

| Field | Value |
|------|--------|
| **Name** | Minecraft AWS GameLift Deploy |
| **Inventory** | `localhost` (You may need to create a simple "localhost" inventory) |
| **Project** | `minecraft-gamelift` (The project from Step 1) |
| **Playbook** | `main.yaml` |
| **Credentials** | `My AWS Credentials` (The AWS credential from Step 2) |

At this point, you can manually run this Job Template from the AAP dashboard to deploy your server. The next section explains how to automate this part.

---

## 🚀 Part 2: Automated Deployment (via GitHub Actions)

This section explains how to automatically trigger the Job Template you just created every time you push code to your `main` branch (GitOps approach).

### Step 1: Get AAP/AWX API Token

GitHub Actions needs a token to authenticate with the AAP/AWX API.

1.  In your AAP/AWX dashboard, go to **Access → Users**.
2.  Select the user you want to generate a token for (a dedicated "service account" user is recommended).
3.  Click the **Tokens** tab and **Add**.
4.  **Type:** `Personal Access Token`
5.  **Scope:** `Write` (This allows the token to launch jobs).
6.  Save and **copy the token immediately**. You will not see it again.

### Step 2: Get Job Template ID

GitHub Actions needs to know *which* template to launch.

1.  In AAP/AWX, go to **Resources → Templates**.
2.  Click on your "Minecraft AWS GameLift Deploy" template.
3.  Look at the URL in your browser. It will look like this:
    `https://your-aap-host.com/#/templates/job_template/12`
4.  The number at the end (e.g., `12`) is your **Job Template ID**. Note it down.

### Step 3: Add GitHub Secrets

Add the AAP token and host information to your GitHub repository's secrets.

1.  Go to your GitHub repository.
2.  Go to **Settings** → **Secrets and variables** → **Actions**.
3.  Click **New repository secret** and add the following three secrets:

    * **Name:** `AWX_HOST`
        * **Value:** Your full AAP/AWX server address (e.g., `https://aap.mycompany.com`)
    * **Name:** `AWX_API_TOKEN`
        * **Value:** The API token you copied in Step 1.
    * **Name:** `AWX_JOB_TEMPLATE_ID`
        * **Value:** The Job Template ID you found in Step 2 (e.g., `12`).

### Step 4: Create GitHub Actions Workflow File

Create a file in your repository at this exact path: `.github/workflows/deploy_minecraft.yml`.

Copy and paste the following content into that file:

```yaml
# .github/workflows/deploy_minecraft.yml

name: Deploy Minecraft to AWS GameLift

# This workflow triggers on a push to the 'main' branch
on:
  push:
    branches:
      - main

jobs:
  trigger-aap-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Launch AAP Job Template
        # We use Red Hat's official Action to launch an AWX/AAP job
        uses: redhat-actions/awx-job-launch@v2
        with:
          # Get the AAP host address from GitHub Secrets
          awx_host: ${{ secrets.AWX_HOST }}
          
          # Get the AAP API token from GitHub Secrets
          awx_token: ${{ secrets.AWX_TOKEN }}
          
          # Get the Job Template ID from GitHub Secrets
          job_template_id: ${{ secrets.AWX_JOB_TEMPLATE_ID }}
          
          # Wait for the job to finish before completing
          wait_for_job_completion: true
          
          # Verbosity level for logging
          job_verbosity: 1
          
          # --- OPTIONAL ---
          # Uncomment the line below ONLY if your AAP server 
          # uses a self-signed SSL certificate.
          # validate_certs: false

### Step 5: Push to Deploy!

Commit and push the new `.github/workflows/deploy_minecraft.yml` file to your `main` branch.

That's it! As soon as you push, you can go to the **Actions** tab in your GitHub repository. You will see the "Deploy Minecraft to AWS GameLift" workflow start. This workflow will securely connect to your AAP/AWX instance and launch the job, which in turn deploys your server to AWS.

---

## 🔧 Cleanup

To avoid AWS costs:
- If a `destroy.yaml` playbook exists, create a new Job Template for it and run it.
- If not, manually clean up the created resources (Fleet, Build, IAM, S3) from the AWS Console.

---

## 💡 Notes

- Currently configured for a **Vanilla Minecraft server**.
- You can add **Paper**, **Spigot**, or **Fabric** builds by modifying the `core.roles.build` section in this repository.
- If you want more advanced integration, this setup can easily be converted into an **Ansible Operator**.

---

## 📚 References

- [AWS GameLift Documentation](https://docs.aws.amazon.com/gamelift/latest/developerguide/)
- [Ansible Automation Platform Docs](https://docs.ansible.com/automation-controller/latest/html/userguide/)
- [Minecraft Dedicated Server Setup](https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server)

# 🕹️ Minecraft Server on AWS GameLift (Ansible Playbook)

This repository contains an **Ansible playbook** to automate the deployment of a **Vanilla Minecraft server** on **AWS GameLift** — Amazon’s managed service for deploying, operating, and scaling dedicated game servers.

---

## 🚀 Overview

This playbook provisions the following AWS resources:
- **IAM Roles & Policies** – for GameLift access and EC2 instance permissions  
- **S3 Bucket** – to store Minecraft server build files and configuration  
- **Build Setup** – prepares the Minecraft server build for GameLift  
- **GameLift Fleet** – deploys and manages the game server infrastructure  
- **Auto Scaling** – enables elastic capacity scaling based on player demand  

The entire process runs locally via **Ansible**, connecting to AWS APIs using your credentials.

---

## 🧩 File Structure

```
main.yaml                   # Main playbook entry point
core/
 ├── vars/main.yaml          # Variable definitions (region, game build, etc.)
 └── roles/
      ├── iam/               # IAM setup tasks
      ├── s3/                # S3 bucket and uploads
      ├── build/             # Minecraft build packaging and upload
      ├── fleet/             # GameLift Fleet creation and configuration
      └── autoscaling/       # Auto scaling policies for GameLift
```

---

## ⚙️ Requirements

Before running this playbook, ensure you have:

- **AWS CLI** configured with admin-level permissions  
- **Ansible** installed (≥ v2.10)  
- **Python 3.x** and **boto3** installed  
- Access to an **AWS account** with GameLift enabled in your desired region  

Install dependencies:
```bash
pip install boto3 botocore ansible
```

---

## 🧠 Usage

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-repo>/minecraft-gamelift.git
   cd minecraft-gamelift
   ```

2. Edit the `core/vars/main.yaml` file and configure:
   ```yaml
   aws_region: eu-central-1
   game_name: "minecraft-vanilla"
   instance_type: "c5.large"
   build_path: "/path/to/minecraft-server"
   ```

3. Run the playbook:
   ```bash
   ansible-playbook main.yaml
   ```

4. Once complete, check your GameLift console:
   - A new **Build** and **Fleet** will appear.
   - Fleet status should eventually move to **Active**.

---

## 🔧 Cleanup

To avoid unnecessary AWS costs:
```bash
ansible-playbook destroy.yaml
```
*(If you have a cleanup playbook; otherwise delete resources via AWS Console.)*

---

## 🛡️ Notes

- The playbook currently deploys a **Vanilla Minecraft server**.  
- You can extend it to support **Spigot**, **Paper**, or **Fabric** builds by modifying the `core.roles.build` role.  
- Ensure your AWS credentials have permissions for:  
  `gamelift:*`, `iam:*`, `ec2:*`, `s3:*`, and `autoscaling:*`.  

---

## 📚 References

- [AWS GameLift Developer Guide](https://docs.aws.amazon.com/gamelift/latest/developerguide/)
- [Ansible AWS Collection](https://docs.ansible.com/ansible/latest/collections/amazon/aws/)
- [Minecraft Server Setup Guide](https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server)

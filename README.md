# Enterprise-Jenkins-on-Azure

Designed and deployed a secure Jenkins Controller-Agent architecture on Microsoft Azure with private networking, SSH key authentication, Network Security Groups (NSGs), and enterprise-grade CI/CD infrastructure practices.



\# 🚀 Azure Jenkins Controller-Agent Secure Lab



!\[Architecture Diagram](docs/architecture-diagram.png)



\---



\## 📖 Project Overview



This project demonstrates an enterprise-style Jenkins deployment on Microsoft Azure using a dedicated Jenkins Controller and a private Jenkins Agent.



The objective is to simulate a real-world DevOps environment where:



\- Jenkins Controller manages CI/CD operations.

\- Jenkins Agent executes build and deployment workloads.

\- Administrative access is restricted using Azure NSGs.

\- SSH key-based authentication is used instead of passwords.

\- Build agents remain private and inaccessible from the Internet.



\---



\## 🏗️ Architecture



```text

Admin Workstation

&#x20;      │

&#x20;      │ HTTP/HTTPS + SSH

&#x20;      ▼

Jenkins Controller VM

(Public Access Restricted)

&#x20;      │

&#x20;      │ SSH over Private Network

&#x20;      ▼

Jenkins Agent VM

(No Public IP)

```



\### Network Flow



```text

Admin Workstation

&#x20;       │

&#x20;       ▼

Jenkins Controller

&#x20;       │

&#x20;       ▼

Private Jenkins Agent

&#x20;       │

&#x20;       ▼

Build / Test / Deploy

```



\---



\## 🎯 Project Goals



\- Deploy Jenkins on Azure

\- Implement secure network segmentation

\- Configure Controller-Agent communication

\- Restrict administrative access

\- Implement SSH Key Authentication

\- Simulate enterprise DevOps infrastructure



\---



\# ☁️ Azure Resources



| Resource | Purpose |

|-----------|----------|

| Resource Group | Resource Management |

| Virtual Network (VNet) | Private Networking |

| Controller Subnet | Hosts Jenkins Controller |

| Agent Subnet | Hosts Jenkins Agent |

| Network Security Group (NSG) | Security Controls |

| Public IP | Controller Access |

| Ubuntu 24.04 VM | Compute Platform |

| Jenkins | CI/CD Automation |

| OpenJDK 21 | Jenkins Runtime |



\---



\# 🔐 Security Design



\## Jenkins Controller Access



| Service | Port | Source |

|----------|------|----------|

| Jenkins Web UI | 8080 | Administrator Public IP |

| SSH | 22 | Administrator Public IP |



\---



\## Jenkins Agent Access



| Service | Port | Source |

|----------|------|----------|

| SSH | 22 | Jenkins Controller |



\---



\## Security Controls Implemented



✅ Restricted Public Access



✅ Private Build Agent



✅ SSH Key Authentication



✅ Azure NSG Filtering



✅ Network Segmentation



✅ Least Privilege Principle



✅ No Public Exposure of Build Nodes



\---



\# 🌐 Azure NSG Rules



\## Controller NSG



| Priority | Name | Port | Protocol | Source | Action |

|-----------|---------|------|----------|---------|---------|

| 200 | allow-jenkins-8080 | 8080 | TCP | Administrator Public IP | Allow |

| 300 | allow-ssh | 22 | TCP | Administrator Public IP | Allow |

| 65000 | AllowVnetInBound | Any | Any | VirtualNetwork | Allow |

| 65001 | AllowAzureLoadBalancerInBound | Any | Any | AzureLoadBalancer | Allow |

| 65500 | DenyAllInBound | Any | Any | Any | Deny |



\---



\## Agent NSG



| Priority | Name | Port | Protocol | Source | Action |

|-----------|---------|------|----------|---------|---------|

| 100 | allow-ssh-from-controller | 22 | TCP | Controller Subnet | Allow |

| 65000 | AllowVnetInBound | Any | Any | VirtualNetwork | Allow |

| 65500 | DenyAllInBound | Any | Any | Any | Deny |



\---



\# 🛠️ Jenkins Installation



\## Step 1 - Update Ubuntu



```bash

sudo apt update

sudo apt upgrade -y

```



\---



\## Step 2 - Install Java 21



```bash

sudo apt install -y openjdk-21-jdk

```



Verify:



```bash

java -version

```



Expected Output:



```text

openjdk version "21"

```



\---



\## Step 3 - Add Jenkins Repository Key



```bash

sudo mkdir -p /etc/apt/keyrings



sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \\

https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

```



Purpose:



\- Verifies package authenticity

\- Prevents installation of tampered packages



\---



\## Step 4 - Add Jenkins Repository



```bash

echo "deb \[signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \\

https://pkg.jenkins.io/debian-stable binary/ | sudo tee \\

/etc/apt/sources.list.d/jenkins.list > /dev/null

```



\---



\## Step 5 - Install Jenkins



```bash

sudo apt update



sudo apt install -y jenkins

```



\---



\## Step 6 - Enable and Start Jenkins



```bash

sudo systemctl enable jenkins



sudo systemctl start jenkins



sudo systemctl status jenkins

```



Expected Result:



```text

active (running)

```



\---



\## Step 7 - Verify Jenkins Port



```bash

sudo ss -tulnp | grep 8080

```



Expected Result:



```text

\*:8080

```



\---



\## Step 8 - Test Jenkins Locally



```bash

curl http://localhost:8080

```



Expected Response:



```text

Authentication required

```



\---



\## Step 9 - Retrieve Initial Admin Password



```bash

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```



\---



\# 🔑 Jenkins Agent Configuration



\## Verify Network Connectivity



From Controller:



```bash

ssh azureuser@<AGENT\_PRIVATE\_IP>

```



Possible Output:



```text

Permission denied (publickey)

```



This indicates:



\- Network connectivity is working

\- SSH authentication is not configured yet



\---



\## Generate SSH Key on Controller



```bash

ssh-keygen -t ed25519

```



Display Public Key:



```bash

cat \~/.ssh/id\_ed25519.pub

```



\---



\## Add Controller Public Key to Agent



Create SSH Directory:



```bash

mkdir -p /home/azureuser/.ssh

```



Add Public Key:



```bash

echo "CONTROLLER\_PUBLIC\_KEY" >> /home/azureuser/.ssh/authorized\_keys

```



Set Ownership:



```bash

chown -R azureuser:azureuser /home/azureuser/.ssh

```



Set Permissions:



```bash

chmod 700 /home/azureuser/.ssh



chmod 600 /home/azureuser/.ssh/authorized\_keys

```



\---



\## Test SSH Authentication



```bash

ssh azureuser@<AGENT\_PRIVATE\_IP>

```



Successful login confirms:



\- Network connectivity

\- SSH configuration

\- Key-based authentication



\---



\# 🖥️ Adding Agent to Jenkins



Navigate to:



```text

Manage Jenkins

→ Nodes

→ New Node

```



Choose:



```text

Permanent Agent

```



\---



\## Agent Configuration



| Setting | Value |

|----------|---------|

| Node Name | linux-agent-01 |

| Remote Root Directory | /home/azureuser/jenkins-agent |

| Labels | linux ubuntu azure |

| Launch Method | Launch agents via SSH |

| Username | azureuser |

| Host | Agent Private IP |



\---



\## Prepare Agent Workspace



```bash

mkdir -p /home/azureuser/jenkins-agent



chown -R azureuser:azureuser /home/azureuser/jenkins-agent

```



\---



\# 🚑 Troubleshooting



\## Check Jenkins Status



```bash

sudo systemctl status jenkins

```



\---



\## Restart Jenkins



```bash

sudo systemctl restart jenkins

```



\---



\## View Jenkins Logs



```bash

sudo journalctl -u jenkins.service -n 100 --no-pager

```



\---



\## Verify Listening Ports



```bash

sudo ss -tulnp

```



\---



\## Verify SSH Connectivity



```bash

ssh azureuser@<AGENT\_PRIVATE\_IP>

```



\---



\## Check Firewall



```bash

sudo ufw status

```



\---



\# 📚 Key Learnings



Through this project I gained hands-on experience with:



\- Jenkins Controller-Agent Architecture

\- Azure Virtual Networking

\- Azure Network Security Groups

\- Linux Administration

\- SSH Key Authentication

\- Enterprise Security Principles

\- CI/CD Infrastructure Design

\- Network Segmentation

\- DevOps Platform Deployment



\---



\# 🚀 Future Improvements



\- Configure HTTPS with SSL Certificates

\- Add DNS Record

\- Deploy Nginx Reverse Proxy

\- Integrate GitHub Webhooks

\- Add Docker Build Agents

\- Build Terraform Deployment Pipelines

\- Integrate Azure Key Vault

\- Configure Monitoring and Alerting

\- Implement Jenkins Backup Strategy

\- Restrict Access Through VPN or Bastion



\---



\# 📸 Recommended Screenshots



Add screenshots for:



\- Azure Architecture

\- Resource Group

\- VNet and Subnets

\- NSG Rules

\- Jenkins Dashboard

\- Jenkins Nodes

\- Agent Connected Status

\- Successful Build Pipeline



\---



\# 🏆 Final Outcome



This project successfully demonstrates:



\- Jenkins deployment on Azure

\- Secure Controller-Agent architecture

\- Private build infrastructure

\- Azure networking implementation

\- NSG security controls

\- SSH key-based authentication

\- Enterprise-style DevOps practices



The resulting environment closely resembles real-world Jenkins deployments used in enterprise cloud environments.



\---



\## 👨‍💻 Author



\*\*Abdulrahman Ibnaof\*\*



Cloud Engineer | DevOps Engineer | Azure Administrator




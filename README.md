# 🛡️ Endpoint Security & Monitoring: Deploying Wazuh SIEM for Real-Time Threat Detection

## 📌 Project Overview
In this hands-on cybersecurity lab, I deployed and configured **Wazuh**, an open-source Enterprise Security monitoring solution. The goal was to establish a centralized **Security Operations Center (SOC)** environment to monitor a Windows 11 endpoint in real-time. 

This project demonstrates practical expertise in:
* **Real-time File Integrity Monitoring (FIM):** Successfully detected and alerted on unauthorized file creations and deletions within sensitive directories.
* **Log Analysis & Authentication Monitoring:** Tracked user authentication patterns, specifically identifying failed logon attempts to monitor for potential unauthorized access.
* **SIEM Dashboards & Custom Filtering:** Utilized **DQL (Dashboards Query Language)** to reduce alert noise and isolate high-priority security threats (Level 7+) for efficient incident response.

---

## 🛠️ Environment & Infrastructure Setup
The lab was built using **Oracle VirtualBox** with a **Bridged Networking** configuration. This was a critical step to ensure seamless communication between the SIEM manager (VM) and the monitored endpoint (Host).

* **SIEM Manager:** Ubuntu Server 22.04 (Running as a VirtualBox VM)
* **Monitored Endpoint:** Windows 11 (Physical Host Machine)
* **Network Mode:** Bridged Adapter (Allows both machines to be on the same subnet)
* **Security Stack:** Wazuh Manager (Server-side), Wazuh Agent (Endpoint-side), DQL (Dashboards Query Language for data analysis)

---

## ⚙️ Phase 1: Wazuh Manager Deployment (Ubuntu Server)
I initiated the Wazuh manager installation on a dedicated Ubuntu server. The installation followed a structured process to ensure security and integrity.

### 🚀 Installation Steps:
1. **GPG Key Integration:** Added the official Wazuh GPG key to verify package authenticity:
   `curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg`

2. **Automated Installation:** Executed the all-in-one installation script to deploy the indexer, server, and dashboard:
   `curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i`

3. **Post-Installation Validation:**
After the process was completed, I identified the Manager's IP as **172.20.10.11**, which was essential for registering the Windows agent in the next phase.

<img width="1919" height="1002" alt="Wazuh Installation Output" src="https://github.com/user-attachments/assets/f5dc8371-c73d-47bf-81ab-c30dc9c8f40c" />
<img width="1919" height="1007" alt="Dashboard Login" src="https://github.com/user-attachments/assets/86ed49ee-9cc6-4709-b51a-d477e10684ef" />

---

## 🛡️ Phase 2: Agent Onboarding via PowerShell

To monitor the Windows host, I deployed the Wazuh Agent using a customized deployment script. This ensures the agent is pre-configured to communicate with the manager.

### 🔑 Step 1: Generating the Deployment Command
I navigated to the **"Deploy new agent"** section in the Wazuh Dashboard and configured the following settings:
* **Operating System:** Windows
* **Wazuh Server Address:** 172.20.10.11
* **Agent Name:** My-Windows-PC

### 🚀 Step 2: Automated Installation & Enrollment
I launched **PowerShell as an Administrator** and executed the generated command. This script automates the download, silent installation, and registration of the agent.

**Command executed:**
`Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.0-1.msi -OutFile ${env:tmp}\wazuh-agent.msi; msiexec.exe /i ${env:tmp}\wazuh-agent.msi /q WAZUH_MANAGER='172.20.10.11' WAZUH_REGISTRATION_SERVER='172.20.10.11' WAZUH_AGENT_NAME='My-Windows-PC'`

### 🏃 Step 3: Starting the Wazuh Service
Once installed, I started the agent service to begin the real-time monitoring:
`NET START WazuhSvc`

*Result: The terminal confirmed: "The Wazuh service was started successfully".*

<img width="1919" height="1006" alt="PowerShell Installation" src="https://github.com/user-attachments/assets/64152ba4-467c-456b-8768-2db562d381dc" />

### ✅ Step 4: Verification & Configuration
I verified the agent appeared as **"Active"** on the dashboard (IP: 172.20.10.3). I then updated the `ossec.conf` file on the Windows host to enable real-time monitoring for the `C:\test` directory.

<img width="1911" height="1003" alt="Screenshot 2026-03-25 130502" src="https://github.com/user-attachments/assets/9b35264a-8cbf-49c1-8065-54781cffec6c" />

<img width="1911" height="1007" alt="Configuration Update" src="https://github.com/user-attachments/assets/e9d3600f-edcd-4e47-a924-4c2b179620ac" />

---

## 🛑 Phase 3: Real-Time Threat Detection & Security Analysis

### 1. File Integrity Monitoring (FIM)
I monitored the `C:\test` directory for unauthorized changes.
* **Activity:** Created and deleted a file named `hacker.txt`.
* **Detection:** Wazuh captured **"File added to the system"** and **"File deleted from the system"** events.
* **Alert Level:** The deletion was flagged as a **Level 7 Alert**.

<img width="1919" height="1007" alt="FIM Alert 1" src="https://github.com/user-attachments/assets/6caf1218-0fea-444a-8174-a608620322b4" />
<img width="1918" height="1016" alt="FIM Alert 2" src="https://github.com/user-attachments/assets/02c3a213-f424-4286-b148-62814e32f370" />

### 2. Authentication & Policy Monitoring
* **Activity:** Performed manual failed login attempts.
* **Observation:** The SIEM captured multiple **"Logon Failure"** and **"User account changed"** alerts (Level 8).

<img width="1919" height="1009" alt="Logon Failures" src="https://github.com/user-attachments/assets/ceeca03e-f7fd-4536-9ab0-c1ec69ae9cba" />

---

## 💡 Troubleshooting & Challenges Faced

* **Network Isolation:** Initially, the Windows Agent could not communicate with the Ubuntu Manager. I resolved this by switching from **NAT** to a **Bridged Adapter** to place both machines on the same subnet.
* **Alert Noise Reduction:** To focus on critical threats, I implemented a custom **DQL filter**: 
  `rule.level >= 7 and not rule.groups: "sca"`
  This successfully isolated high-priority security threats for efficient analysis.

<img width="1919" height="1006" alt="DQL Filtering" src="https://github.com/user-attachments/assets/14ec2e39-6880-4b75-8ca0-a022a9c19c6d" />

---

## 🏁 Conclusion
This project reinforced the importance of centralized visibility. By deploying Wazuh, I gained practical experience in endpoint integrity monitoring, log analysis, and noise reduction using DQL, proving the value of SIEM in modern cybersecurity operations.

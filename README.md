# SAP BTP Integration Suite — Automated SFTP File Delivery
### From Local PC to Google Cloud SFTP Server via SAP CPI (Trial Account)

---

## Overview

This repository contains a SAP BTP Integration Suite iFlow that automates the secure delivery of a file from a local PC to a remote SFTP server hosted on Google Cloud. The entire pipeline runs fully automated — no manual intervention required once deployed.

Built and tested on a **SAP BTP Trial Account** using only third-party tools and SAP Cloud Platform Integration (CPI).

---

## Architecture

```
Local PC (Sender)
     │
     │  SFTP Adapter (inbound)
     ▼
SAP Integration Suite — Integration Process
     │
     ├── Start Event
     ├── Content Modifier 1  (sets message body & headers)
     └── End Event
     │
     │  SFTP Receiver Adapter (outbound / SSH-encrypted)
     ▼
Internet — Fixed Egress IP (allowlisted on firewall)
     │
     ▼
Linux VM — Google Cloud
     │
     ├── SFTP Daemon (sshd) — authenticates & accepts files
     └── SFTP User Directory — /home/sftpuser
     │
     ▼
File delivered to SFTP server ✅
```

---

## Step-by-Step Setup

### 1. Create SFTP Server on Google Cloud
- Deploy a Linux VM on Google Cloud
- Install and configure an SFTP server (`openssh-server`)
- Create a dedicated SFTP user, password, and target folder
- Note the VM's **external IP address**

### 2. Get SFTP Connection Details
- **Host / IP** — the external IP of your Google Cloud VM
- **Port** — `22` (default SSH/SFTP port)
- **Username & Password** — the SFTP user created in Step 1

### 3. Get the Host Key (required for SAP CPI)
- On the SFTP server, copy the host key from `/etc/ssh`
- Create a `known_hosts` file containing the host key entry

### 4. Upload Host Key to SAP CPI Trial
- In SAP Integration Suite go to:
  > **Operations View → Manage Security → Security Material**
- Upload the `known_hosts` file and deploy it

### 5. Create User Credentials in Security Material
- Still in **Security Material**, add a new **User Credentials** entry
- Enter the SFTP **username and password** from Step 2
- Note the credential name — you will need it in the iFlow configuration

### 6. Create the iFlow in SAP CPI
- Create a new integration package
- Add a new iFlow with the following components:
  - **SFTP Sender Adapter** — connects to the local PC / source
  - **Content Modifier** — sets the message body and target file name
  - **SFTP Receiver Adapter** — configure with:
    - Host (VM IP)
    - Port (`22`)
    - Directory (`/home/sftpuser`)
    - Credential Name (from Step 5)

### 7. Deploy and Test
- Deploy the iFlow from the Integration Suite designer
- Send a test payload
- Verify the file appears in `/home/sftpuser` on the Google Cloud VM ✅

---

## Security Notes
- The Google Cloud VM firewall is configured to **allowlist the SAP BTP egress IP** only
- All file transfers are **SSH-encrypted** end to end
- SFTP credentials are stored securely in **SAP Security Material** — never hardcoded
- Sensitive details (full IP address, instance ID) have been redacted in public documentation

---

## Technologies Used

| Technology | Purpose |
|---|---|
| SAP BTP Integration Suite (CPI) | Integration platform & iFlow execution |
| SAP SFTP Receiver Adapter | Outbound secure file transfer |
| Google Cloud VM (Linux) | Target SFTP server |
| OpenSSH / sshd | SSH authentication & file acceptance |
| known_hosts | Host key verification |
| SAP Security Material | Secure credential storage |

---

## Repository Contents

| File | Description |
|---|---|
| `sftp_google.zip` | Exported SAP iFlow package — importable into any SAP CPI tenant |
| `README.md` | This documentation file |

---

## How to Import the iFlow

1. Download `sftp_google.zip` from this repository
2. Go to **SAP Integration Suite → Design**
3. Open or create an integration package
4. Click **"Import"** and select the `.zip` file
5. Configure the SFTP connection details for your environment
6. Upload your own `known_hosts` and credentials to Security Material
7. Deploy and test

---

## Author

**Raul Planas**
SAP Integration Developer · SAP BTP · iFlow · SFTP · Cloud Networking

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://www.linkedin.com/in/)

---

## Licence

This project is shared for educational and demonstration purposes.


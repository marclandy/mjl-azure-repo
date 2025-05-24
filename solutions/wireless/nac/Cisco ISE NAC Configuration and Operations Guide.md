---
layout: post
title: "Cisco ISE NAC Configuration and Operations Guide"
date: 2025-05-24
author: Marc Landy
categories: [enterprise, design, cisco ise, wireless]
tags: [cisco ise]
---

---
layout: post
title: "Cisco ISE NAC Configuration and Operations Guide"
date: 2025-05-24
categories: [Cisco, ISE, NAC, Zero Trust]
tags: [ISE, NAC, Cisco, Wireless, BYOD, Guest, Security]
author: NetworkOps
---

<style>
  h2 { border-bottom: 1px solid #ccc; padding-bottom: 4px; }
  code { background-color: #f5f5f5; padding: 2px 4px; border-radius: 4px; }
  pre { background-color: #f9f9f9; padding: 10px; border-left: 4px solid #007acc; overflow-x: auto; }
</style>

# Cisco ISE NAC Configuration and Operations Guide

## Configuration Guide

Cisco ISE NAC Operational Guide

1. Runbook: Cisco ISE NAC Day 0, Day 1, and Day 2 Tasks

Day 0 – Planning and Initial Configuration

Day 1 – Service and Policy Deployment

Day 2 – Operational Maintenance and Monitoring

ISE System Backup

When GUI Exports Are Preferred:

Backups: Full system backups, configuration exports (like network devices, endpoint identity groups, policies).

Reports: Compliance reports, RADIUS live logs, MDM posture details, pxGrid activity.

Policy Export/Import: Downloading rule sets or policy elements for migration or documentation.

Bulk Operations: Using CSVs to import/export endpoints, users, or device profiles.

When CLI Exports Are Preferred:

Tech Support: Generating show tech-support or collect logs bundles.

Diagnostics: Running show, debug, or ping commands to troubleshoot.

Configuration Audit: Exporting network devices, trusted certificates, profiling settings via CLI.

Automation: Used with scripts/Ansible to extract specific config elements.

1. Full System Backups (GUI-based)

Performed from:

Administration > System > Backup & Restore

GUI Steps:

Go to Backup & Restore.

Click "Backup Now".

Choose Backup Type:

Configuration Backup – Includes system config, network devices, policies.

Operational Backup – Includes logs, reports, session data (used for MnT nodes).

Select the repository (FTP, SFTP, etc.).

Click "OK".

📁 Output Format: .gpg encrypted archive (e.g., iseConfigBackup-2025-05-11.gpg)

2. Define Repositories for Backups

Before backups can be scheduled or exported, define a repository:

Administration > System > Maintenance > Repository

Example – SFTP Repository Setup:

Name: SFTPRepo01

Protocol: SFTP

Server: 10.0.0.10

Path: /ise\_backups/

Username/Password: backupuser / ********

3. CLI Backup Commands (ISE CLI)

backup iseConfigBackup config repository SFTPRepo01

backup iseOpBackup operational repository SFTPRepo01

To schedule:

conf t

scheduler job name DailyConfigBackup

backup iseConfigBackup config repository SFTPRepo01

scheduler schedule name DailyConfigSchedule job DailyConfigBackup

periodic daily at 2:00

4. Exporting Specific Configs (GUI CSV Exports)

From GUI:

5. Exporting Certificate Trust Lists

GUI Path:

Administration > System > Certificates > Trusted Certificates

Click each cert > Export (Base64 or DER)

6. Monitoring Data Backup (MnT Node)

Operational backup must include Monitoring node data.

CLI:

backup iseMnTBackup operational repository SFTPRepo01

7. Restore Process (GUI or CLI)

GUI:

Administration > System > Backup & Restore > Restore

CLI:

restore iseConfigBackup config repository SFTPRepo01 passphrase MySecret123

Security Considerations:

Always encrypt and store backups offsite.

Backups contain sensitive info like PSKs, EAP certs, shared secrets.

Use role-based access and protect CLI access with TACACS+.

Cisco ISE CLI Export Snippets

Here are commonly used CLI commands for an ISE specialist:

# Export running configuration

show running-config

# Export trusted certificates

show crypto pki certificates

# Export all network devices

show network access device

# Export endpoint profiles

show profiling policies

# Export authorization/authentication policy names

show policy-set

# Export live sessions

show user-session all

# Generate and download tech support logs

tech-support all

repository disk

collect-logs

Optional: Export via Repository

First, define a repository (FTP, SFTP, TFTP):

configure terminal

repository backupRepo

url sftp://10.10.10.10/ise\_backup

user iseadmin password plain iseP@ssword

exit

Then export logs or backups:

backup ise-config backupRepo

collect-logs repository backupRepo

2. Ansible Playbook: ISE Configuration and Operational Backup

Ansible playbook that automates Cisco ISE CLI backups via SSH, leveraging a preconfigured repository (e.g., SFTPRepo01) on ISE.

This script assumes:

You have SSH access to Cisco ISE (admin CLI)

The ISE node has a repository configured (e.g., SFTPRepo01)

You're using Ansible with paramiko or ssh connection

The ISE node is accessible by hostname or IP

ise\_backup.yml — Ansible Playbook for ISE Config & Operational Backup

yaml

---

- name: Cisco ISE Backup Automation

hosts: ise\_nodes

gather\_facts: no

connection: network\_cli

vars:

backup\_repository: "SFTPRepo01"

config\_backup\_name: "iseConfigBackup"

operational\_backup\_name: "iseOpBackup"

tasks:

- name: Run Configuration Backup

ios\_command:

commands:

- "backup {{ config\_backup\_name }} config repository {{ backup\_repository }}"

register: config\_backup\_output

- name: Print Config Backup Output

debug:

var: config\_backup\_output.stdout\_lines

- name: Run Operational Backup

ios\_command:

commands:

- "backup {{ operational\_backup\_name }} operational repository {{ backup\_repository }}"

register: operational\_backup\_output

- name: Print Operational Backup Output

debug:

var: operational\_backup\_output.stdout\_lines

inventory.ini — Inventory for the ISE node(s)

[ise\_nodes]

ise01.example.com ansible\_user=admin ansible\_password=ISE\_Admin\_Pass ansible\_network\_os=ios ansible\_connection=network\_cli

Prerequisites

Install the necessary Ansible collections if not already available:

ansible-galaxy collection install cisco.ios

Or for legacy ISE CLI (if needed, try raw SSH):

yaml

connection: ssh

ansible\_shell\_type: cisco

ansible\_network\_os: cisco.ios.ios

Optional: Run It on a Schedule

Use cron or a job scheduler to trigger the playbook periodically:

bash

crontab -e

cron

CopyEdit

0 2 * * * /usr/bin/ansible-playbook /path/to/ise\_backup.yml -i /path/to/inventory.ini

Security Notes

Avoid plaintext passwords — use Ansible Vault for secrets.

Protect the SSH key or credentials with strict file permissions.

3. ISE CLI Output: Policy Sets

802.1X is a Port Based Network Access Control, defining 3 roles:

Supplicant (station, client device),

Authenticator (AP or WLC) and

Authentication Server (RADIUS).

Extensible Authentication Protocol (EAP) is the authentication framework supporting multiple methods such as PEAP, EAP-TLS, EAP-TTLS & more. It’s a datalink layer protocol, IP is not required. Additionally, Authenticator does not have to understand the authentication method.

RADIUS carries AAA information between Authentication and RADIUS Server.

Supplicant and Authenticator use EAPOL in wireless to exchange authentication data.

Authenticator and Authentication Server talk over RADIUS.

Both parts (EAPOL + RADIUS) form an authentication mechanism called 802.1X.

Let’s see step by step what happens in the 802.1X EAP process:

Open System Authentication:

First the client and the AP go through 802.11 Open System Authentication, that is made up of 2 frame exchanges – client sends open auth to the AP & then the AP responds with open auth success.

802.11 Association:

Next in the frame exchange is 802.11 Association, this is also 2 frame exchanges – client sends association request to the AP & then the AP responds with an association response.

802.1x EAP Authentication (below is based on EAP-TLS, but it will be similar for other EAP methods):

Now we move on to the juicy part of the frame exchanges – “802.1X EAP authentication”. The first frame in this exchange is from the client which sends an “EAPOL start message” to the AP to start EAP authentication.

The client is then asked for its identity in an “EAP Request/Identity” message from the AP.

The client replies with an “EAP Response/Identity” message with its (dummy) user ID and the request to use TLS, which is forwarded to the RADIUS server.

The RADIUS server, upon receiving the RADIUS access request & RADIUS access challenge (EAP Response/Identity message), starts the server-side TLS process by sending an EAP-TLS Start message to the client.

The client responds with an EAP response – client hello message.

The RADIUS server replies with an EAP Request message— a TLS server hello. It provides its certificate to the client, TLS protocol version, a cipher suite, and the client requests the certificate.

The client validates the server certificate and responds with an EAP Response message that contains its certificate. This message starts the negotiation for cryptographic specifications – the cipher and compression algorithms.

After the client certificate is validated, the RADIUS server responds with cryptographic specifications for the session.

The client responds with an EAP-Response packet of EAP-Type = EAP-TLS with no data, notifying the RADIUS server that it has received the cryptographic specifications.

The RADIUS server sends an EAP-Success message to the AP indicating successful authentication.

The RADIUS server creates the session Master Key, also known as the PMK (Pairwise Master Key).

The client also creates the PMK.

4-Way Handshake:

The client and the AP run the 4-way handshake to create the session keys. Which are:

EAPOL Key Packet No.1(Authenticator Nonce) – Client calculated PTK

EAPOL Key Packet No.2 (Supplicant Nonce, MIC) – Authenticator calculated PTK

EAPOL Key Packet No.3 (Install PTK, MIC, Encrypted GTK)

Now we have the GTK (Group Temporal Key) encrypted in the PTK.

EAPOL Key Packet No. 4 (MIC)

Voila! We now have fully established an encrypted 802.1X EAP-TLS session!

We have also made a diagram of the process so you can visualise the above a bit easier!

---

# End of Guide

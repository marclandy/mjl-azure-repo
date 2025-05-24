---
layout: post
title: "Cisco ISE NAC Configuration and Operations Guide"
date: 2025-05-24
author: Marc Landy
categories: [enterprise, design, cisco ise, wireless]
tags: [cisco ise]
---
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

---

# End of Guide

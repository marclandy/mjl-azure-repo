# Cisco ISE NAC Configuration and Operations Guide

A comprehensive guide for configuring and operating Cisco ISE (Identity Services Engine) with Network Access Control (NAC) for **Cisco ISE**, **Meraki**, **Intune**, and mixed **Centralised/Distributed** wireless models.

## Table of Contents

- [ISE Configuration](#ise-configuration)
- [RADIUS Server Settings](#radius-server-settings)
- [Certificate Requirements](#certificate-requirements)
- [Active Directory Configuration](#active-directory-configuration)
- [Intune Configuration](#intune-configuration)
- [Network Architecture](#network-architecture)
- [Meraki SSID Configuration](#meraki-ssid-configuration)
- [Operations Guide](#operations-guide)
- [Backup and Export Procedures](#backup-and-export-procedures)
- [Automation with Ansible](#automation-with-ansible)
- [802.1X Authentication Process](#8021x-authentication-process)

## ISE Configuration

### ISE Services

Enable the following core services:
- **802.1X Authentication**
- **Profiling**
- **Posture Assessment**
- **TrustSec Segmentation**

### Authentication Policy

```
IF Wireless_802.1X AND EAP-TLS
THEN Use Identity Source: Active Directory

ELSE IF Wireless_802.1X AND PEAP-MSCHAPv2
THEN Use Identity Source: Active Directory

ELSE IF MAB
THEN Use Internal Endpoints Database
```

### Authorization Policy

```
IF AD-Group = "Corp-Users" AND EAP-TLS
THEN PermitAccess, VLAN 10, DACL:PermitAll, TrustSec_SGT:Corp

IF AD-Group = "BYOD-Users" AND PEAP-MSCHAPv2 AND Posture = Compliant
THEN PermitAccess, VLAN 20, DACL:InternetOnly, TrustSec_SGT:BYOD

IF AD-Group = "BYOD-Users" AND Posture = Non-Compliant
THEN Quarantine VLAN, DACL:DenyAll

IF EndpointProfile = "IoT-Device"
THEN PermitAccess, VLAN 40, DACL:Restricted

IF Wireless SSID = "GUEST"
THEN GuestFlow via Captive Portal, VLAN 30
```

### Profiling & Posture

- Enable DHCP, RADIUS, and HTTP Profiling probes
- Define profiles for:
  - Windows/macOS
  - iOS/Android
  - IoT Cameras, Printers
- Enable posture checks (AV status, disk encryption, OS compliance) via AnyConnect/NAC agent

## RADIUS Server Settings

### Meraki Dashboard → RADIUS Configuration

**RADIUS Server:**
- Name: ISE-Cluster-1
- IP: 10.0.10.10
- Port: 1812
- Secret: `<shared_secret>`
- NAS ID: CORP-WiFi

**RADIUS Accounting:**
- Port: 1813

Configure the same for **BYOD** and **GUEST** SSIDs. Ensure **RADIUS CoA** is enabled on Meraki for posture rechecks or role changes.

## Certificate Requirements

### Internal PKI (or 3rd-party)

- Issue client certs via Intune/SCEP or GPO
- Root CA trusted by ISE and all supplicants

### Certificate Templates

- **Client Authentication**: with CN = hostname, SAN = UPN/email
- **ISE Server Certificate**: CN = ise.company.com (wildcard or SAN-based for clusters)

### EAP-TLS

- Mandatory for CORP-WiFi
- Verify certs using CRL or OCSP
- Enable **certificate-based machine and user auth**

## Active Directory Configuration

### Organizational Units & Groups

- OU=Corporate Devices
- OU=BYOD Devices
- OU=IoT
- OU=GuestUsers

### AD Groups (used in ISE Policies)

- Corp-Users
- BYOD-Users
- ISE-Admins

### Group Policy for Cert Enrollment

- Auto-enroll via GPO for machine + user certs
- Push WLAN profile with SSID = CORP-WiFi, auth = EAP-TLS

## Intune Configuration

### Intune Configuration

- **SCEP Connector** to issue certs to enrolled devices
- **Compliance Policies**:
  - Require BitLocker/FileVault
  - Require AV
  - Minimum OS version
- **Conditional Access**: Integration with Azure AD & ISE via Intune connector

### Device Tagging for ISE

- Devices marked as compliant via Intune can be tagged in ISE as Compliant=True
- Used in **Authorization policy** for BYOD VLAN 20 or quarantine fallback

## Network Architecture

### Centralised Architecture (e.g., University/Enterprise Campus)

- **Topology**: MR APs → Campus Core → ISE → AD/Intune → Internet
- **Roaming**: Enable 802.11r/k/v
- **L3 Roaming**: Set up Mobility Anchors (Catalyst/Cisco/Meraki config)
- **DHCP & DNS**: Centralized DHCP with option 82
- **QoS**:
  - WMM Voice profile on voice SSIDs
  - DSCP to Queue Mapping on Catalyst switches
- **VLANs**: All VLANs trunked across campus and available to APs/controllers

### Distributed Architecture (e.g., Retail Sites)

- **Topology**: MR APs → Cloud → RADIUS Proxy (e.g., Portnox, SecureW2)
- **Authentication**:
  - Use EAP-TLS or PSK + RADIUS-based MAC auth
  - Local breakout (no central tunnel)
- **Cloud PKI/NAC**: SecureW2 or Cloud ISE Lite
- **VLANs**:
  - Local VLANs per SSID: 10 (Corp), 20 (BYOD), 30 (Guest), 40 (IoT)
- **Group Policies**:
  - Apply Meraki Group Policies based on RADIUS attributes (e.g., Filter-ID)

## Meraki SSID Configuration

### CORP-WiFi (802.1X with EAP-TLS)

1. Navigate to **Wireless > Configure > Access control**
2. Select the **CORP-WiFi** SSID
3. Under **Network access**, choose **Enterprise with my RADIUS server**
4. In the **RADIUS servers** section:
   - Click **Add a server**
   - Enter the RADIUS server's IP address, port (default is 1812), and the shared secret
5. Under **Advanced settings**:
   - Set **WPA encryption mode** to **WPA2 only** or **WPA3** as per your security requirements
   - Enable **802.11r** for fast roaming
   - Configure VLAN tagging if dynamic VLAN assignment is used

### BYOD-Secure (802.1X with PEAP-MSCHAPv2)

1. Navigate to **Wireless > Configure > Access control**
2. Select the **BYOD-Secure** SSID
3. Under **Network access**, choose **Enterprise with my RADIUS server**
4. Add your RADIUS server details as above
5. Under **Advanced settings**:
   - Set **WPA encryption mode** to **WPA2 only**
   - Enable **802.11r** for fast roaming
   - Configure VLAN tagging for BYOD VLAN

### Guest (Captive Portal)

1. Navigate to **Wireless > Configure > Access control**
2. Select the **Guest** SSID
3. Under **Network access**, choose **Open (no encryption)**
4. Under **Splash page**, select **Sign-on with my RADIUS server**
5. Add your RADIUS server details
6. Configure walled garden settings to allow access to necessary domains before authentication

### IoT (MAC Authentication Bypass - MAB)

1. Navigate to **Wireless > Configure > Access control**
2. Select the **IoT** SSID
3. Under **Network access**, choose **Enterprise with my RADIUS server**
4. Add your RADIUS server details
5. Enable **MAC-based access control**
6. Configure VLAN tagging for IoT devices

## Operations Guide

### Day 0 - Planning and Initial Configuration

| Task | Description | Tools/Interfaces |
|------|-------------|------------------|
| Define NAC Policy Requirements | Document access policies (802.1X, MAB, Guest, Posture) and business rules per user/device type | Design Docs, Workshops |
| Network Discovery & Classification | Inventory switches, wireless controllers, and IP ranges for ISE integration | IPAM, CMDB |
| Integrate with Active Directory | Configure AD identity source in ISE with join and test | ISE Admin GUI: *Administration > Identity Management > External Identity Sources > AD* |
| Configure Network Devices | Add NADs (switches, WLCs) with RADIUS shared secrets | *Administration > Network Resources > Network Devices* |
| Install Trusted Certificates | Install root/intermediate CA certs and generate ISE node certs for EAP/HTTPS | CLI / GUI: *Administration > System > Certificates* |
| Configure RADIUS and TACACS Services | Enable on PSNs; define ports, timeouts | *Administration > Deployment > Nodes* |
| Enable Logging to Syslog & SNMP | Define remote syslog destinations and SNMP traps | *Administration > System > Logging / SNMP* |
| Time & NTP Configuration | Ensure ISE and NADs sync to trusted NTP | CLI: show clock, ntp server x.x.x.x |
| Licensing | Register smart licenses and activate features like Plus/Apex | *Administration > System > Licensing* |

### Day 1 - Service and Policy Deployment

| Task | Description | Tools/Interfaces |
|------|-------------|------------------|
| Define Policy Sets | Create policy sets for 802.1X wired/wireless, MAB, Guest, Posture | *Policy > Policy Sets* |
| Role-Based Access (RBAC) | Define internal/admin users and assign roles | *Administration > Admin Access* |
| Device Profiling Setup | Enable profiling probes (DHCP, SNMP, HTTP) and categorize endpoints | *Work Centers > Profiler* |
| Configure VLAN/SGT Assignment | Define downloadable ACLs or SGTs for segmentation | *Policy > Results > Authorization* |
| Test Authentication | Validate endpoint access (EAP-TLS, PEAP, MAB fallback) | ISE Live Logs, Switch Debugs |
| Posture & Agent Configuration | Install NAC Agent (Cisco AnyConnect/Posture), enable posture policies | *Work Centers > Posture* |
| Guest Portal Setup | Configure self-registration, sponsor approval, and terms of use | *Work Centers > Guest Access* |
| Certificate Enrollment (SCEP/Manual) | Setup device trust using GPO/SCEP/ISE internal CA if used | GPO, ISE Internal CA |
| Log Monitoring | Monitor authentication and authorization logs for errors and anomalies | *Operations > RADIUS > Live Logs* |

### Day 2 - Operational Maintenance and Monitoring

| Task | Description | Tools/Interfaces |
|------|-------------|------------------|
| Backup Configuration & Certificates | Schedule configuration backups and export node certs | GUI: *Administration > System > Backup and Restore* or CLI |
| Patch and Software Updates | Apply hotfixes and maintenance releases (e.g., ISE 3.x patch bundles) | CLI + GUI |
| Endpoint Cleanup | Purge stale endpoints after retention threshold | *Administration > Identity Management > Settings* |
| Review Logs and Reports | Daily/weekly review of authentication failures, threats, and trends | *Operations > Reports* |
| Policy Tuning | Adjust policy sets based on access trends, posture compliance failures | Policy GUI |
| Certificate Expiry Monitoring | Monitor for expiring internal or 3rd-party certificates | *Administration > System > Certificates* |
| High Availability Health Check | Validate PAN/PSN/MnT node sync and redundancy | CLI: show application status ise |
| Integration Review | Ensure integrations with AD, MDM, SIEM are operating correctly | AD/Intune/Syslog Status |
| User Support / Helpdesk Escalations | Troubleshoot endpoint issues (e.g., supplicant misconfigs, auth failures) | Live Logs, NAD logs, TAC |
| Compliance Audit Readiness | Generate compliance reports and policy audit trails | *Operations > Reports > Compliance* |

## Backup and Export Procedures

### Full System Backups (GUI-based)

Performed from: `Administration > System > Backup & Restore`

**GUI Steps:**
1. Go to Backup & Restore
2. Click **"Backup Now"**
3. Choose **Backup Type**:
   - **Configuration Backup** - Includes system config, network devices, policies
   - **Operational Backup** - Includes logs, reports, session data (used for MnT nodes)
4. Select the **repository** (FTP, SFTP, etc.)
5. Click **"OK"**

📁 Output Format: .gpg encrypted archive (e.g., iseConfigBackup-2025-05-11.gpg)

### CLI Backup Commands

```bash
backup iseConfigBackup config repository SFTPRepo01
backup iseOpBackup operational repository SFTPRepo01
```

To schedule:
```bash
conf t
scheduler job name DailyConfigBackup
backup iseConfigBackup config repository SFTPRepo01
scheduler schedule name DailyConfigSchedule job DailyConfigBackup
periodic daily at 2:00
```

### Exporting Specific Configs

| What | Location | Format |
|------|----------|---------|
| **Network Devices** | Administration > Network Resources > Network Devices | CSV |
| **Endpoint Identity Groups** | Work Centers > Identity Services > Endpoints | CSV |
| **Policy Sets** | Policy > Policy Sets > Export | XML |
| **Profiling Policies** | Work Centers > Profiler > Profiling Policies | GUI only |

### Common CLI Export Commands

```bash
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
```

## Automation with Ansible

### ISE Backup Ansible Playbook

```yaml
---
- name: Cisco ISE Backup Automation
  hosts: ise_nodes
  gather_facts: no
  connection: network_cli
  vars:
    backup_repository: "SFTPRepo01"
    config_backup_name: "iseConfigBackup"
    operational_backup_name: "iseOpBackup"
  
  tasks:
    - name: Run Configuration Backup
      ios_command:
        commands:
          - "backup {{ config_backup_name }} config repository {{ backup_repository }}"
      register: config_backup_output
    
    - name: Print Config Backup Output
      debug:
        var: config_backup_output.stdout_lines
    
    - name: Run Operational Backup
      ios_command:
        commands:
          - "backup {{ operational_backup_name }} operational repository {{ backup_repository }}"
      register: operational_backup_output
    
    - name: Print Operational Backup Output
      debug:
        var: operational_backup_output.stdout_lines
```

### Inventory File (inventory.ini)

```ini
[ise_nodes]
ise01.example.com ansible_user=admin ansible_password=ISE_Admin_Pass ansible_network_os=ios ansible_connection=network_cli
```

### Prerequisites

Install the necessary Ansible collections:
```bash
ansible-galaxy collection install cisco.ios
```

### Schedule with Cron

```bash
crontab -e
# Add this line for daily backups at 2 AM
0 2 * * * /usr/bin/ansible-playbook /path/to/ise_backup.yml -i /path/to/inventory.ini
```

## 802.1X Authentication Process

802.1X is a Port Based Network Access Control, defining 3 roles:
- **Supplicant** (station, client device)
- **Authenticator** (AP or WLC)
- **Authentication Server** (RADIUS)

![wifininjas.net](https://github.com/marclandy/enterprise-infra/blob/eb6bba75f678a06f36367962125945b6b234aa8d/solutions/wireless/nac/images/802.1X-EAP-and-802.11-Security-Keys-Generation-Process_wifininja.jpg)

Extensible Authentication Protocol (EAP) is the authentication framework supporting multiple methods such as PEAP, EAP-TLS, EAP-TTLS & more. It's a datalink layer protocol, IP is not required. Additionally, Authenticator does not have to understand the authentication method.

### Step-by-Step 802.1X EAP Process

#### Open System Authentication
1. First the client and the AP go through 802.11 Open System Authentication, that is made up of 2 frame exchanges -- client sends open auth to the AP & then the AP responds with open auth success.

#### 802.11 Association
2. Next in the frame exchange is 802.11 Association, this is also 2 frame exchanges -- client sends association request to the AP & then the AP responds with an association response.

#### 802.1x EAP Authentication (EAP-TLS example)
3. Now we move on to the juicy part of the frame exchanges -- "802.1X EAP authentication". The first frame in this exchange is from the client which sends an "EAPOL start message" to the AP to start EAP authentication.

4. The client is then asked for its identity in an "EAP Request/Identity" message from the AP.

5. The client replies with an "EAP Response/Identity" message with its (dummy) user ID and the request to use TLS, which is forwarded to the RADIUS server.

6. The RADIUS server, upon receiving the RADIUS access request & RADIUS access challenge (EAP Response/Identity message), starts the server-side TLS process by sending an EAP-TLS Start message to the client.

7. The client responds with an EAP response -- client hello message.

8. The RADIUS server replies with an EAP Request message--- a TLS server hello. It provides its certificate to the client, TLS protocol version, a cipher suite, and the client requests the certificate.

9. The client validates the server certificate and responds with an EAP Response message that contains its certificate. This message starts the negotiation for cryptographic specifications -- the cipher and compression algorithms.

10. After the client certificate is validated, the RADIUS server responds with cryptographic specifications for the session.

11. The client responds with an EAP-Response packet of EAP-Type = EAP-TLS with no data, notifying the RADIUS server that it has received the cryptographic specifications.

12. The RADIUS server sends an EAP-Success message to the AP indicating successful authentication.

13. The RADIUS server creates the session Master Key, also known as the PMK (Pairwise Master Key).

14. The client also creates the PMK.

#### 4-Way Handshake
15. The client and the AP run the 4-way handshake to create the session keys:
    - EAPOL Key Packet No.1(Authenticator Nonce) -- Client calculated PTK
    - EAPOL Key Packet No.2 (Supplicant Nonce, MIC) -- Authenticator calculated PTK
    - EAPOL Key Packet No.3 (Install PTK, MIC, Encrypted GTK)
    - Now we have the GTK (Group Temporal Key) encrypted in the PTK
    - EAPOL Key Packet No. 4 (MIC)

16. Voila! We now have fully established an encrypted 802.1X EAP-TLS session!

## Security Considerations

- Always **encrypt** and **store backups offsite**
- Backups contain **sensitive info** like PSKs, EAP certs, shared secrets
- Use **role-based access** and protect CLI access with TACACS+
- Avoid plaintext passwords — use **Ansible Vault** for secrets
- Protect SSH keys or credentials with strict file permissions

## References

- [WiFi Ninja Blog - 802.1X EAP](https://wifininjas.net/2020/01/10/wn-blog-026-802-1x-eap/)
- Meraki Documentation: Configuring RADIUS Authentication with WPA2-Enterprise

---

*This guide provides a comprehensive overview of Cisco ISE NAC configuration and operations. Always refer to the latest Cisco documentation for the most up-to-date procedures and best practices.*

# Microsoft Intune

Part of Endpoint Manager & O365 ecosystem

## Overview

My primary interest relates to *device management, network enforcement, compliance integration, conditional access*, *and policy enforcement across hybrid environments*.

## Relevance to Network & Infrastructure Architects:

| **Area** | **How Intune Applies** |
|----------|------------------------|
| **Wi-Fi/802.1x** | Pushes enterprise-grade Wi-Fi and certificate configs. |
| **VPN** | Enables secure remote access via per-app or full-device VPNs. |
| **NAC / Zero Trust** | Enforces access based on device compliance, risk, and identity posture. |
| **Cloud-native Management** | Replaces GPO and SCCM with modern, scalable MDM/MAM. |
| **BYOD Strategy** | Secure corporate apps without full device management. |
| **Remote Operations** | Enables remote wipe, re-provisioning, and compliance recovery. |
| **Security Baselines** | Align device configurations with security hardening guides (CIS/NIST). |

## Conditional Access & Network Enforcement (in conjunction with Entra ID / Azure AD)

**Policy-based access control**: Enforce **user + device + app + location + risk**-based access policies.

**Require compliant or hybrid Azure AD-joined devices** for access to corporate resources.

**Integration with Microsoft Defender for Endpoint** to enforce network access based on device risk.

**Network-aware conditional access**: Can enforce **trusted network** or **trusted IP ranges**.

**VPN and Wi-Fi profiles**: Centrally push **Wi-Fi SSIDs**, **802.1x certificates**, and **per-app VPN** configurations.

## Device Lifecycle Management (Windows, macOS, iOS, Android)

**Auto-enrollment** via Azure AD Join / Windows Autopilot.

**Zero Touch Provisioning** (ZTP) for Windows Autopilot, Apple ADE, and Android Enterprise.

**Remote actions**: Restart, lock, wipe, retire, locate, sync.

**Device compliance policies**: Define health, encryption, OS version, rooted/jailbroken status.

## Endpoint Configuration & Policy Enforcement

**Configuration profiles** for security baselines, firewall settings, password rules, and more.

**Endpoint Security policies**: BitLocker, AV/EDR, firewall, attack surface reduction.

**Group policy migration tool**: Convert legacy GPOs to Intune-compatible profiles.

**PowerShell scripts and Win32 app deployments** for deep customization.

## Network and Connectivity Configuration

**Wi-Fi profiles with certificate-based auth** (EAP-TLS, PEAP, etc.).

**VPN profiles**: Supports multiple vendors (Cisco, F5, Zscaler, Palo Alto, etc.).

**Per-app VPN** configuration for secure app-specific tunnels on iOS/macOS.

**Proxy settings** and **network isolation** controls (e.g., via Windows Information Protection).

Integration with **NPS / RADIUS** for 802.1x enforcement.

## Application Management (Including MAM without enrollment)

**Managed app policies** (Intune App Protection) for Outlook, Edge, Teams, etc.

**App VPNs and data leakage protection** for unmanaged BYO devices.

**Block copy/paste, save-as, screen capture**, etc.

**Deploy and manage Win32, MSI, UWP, iOS, and Android apps**.

## Endpoint Security & Zero Trust Integration

Native integration with:

- **Microsoft Defender for Endpoint** (device risk signals).
- **Microsoft Purview** (data loss prevention).
- **NAC integrations**: Cisco ISE, F5 APM via compliance API.

Supports **Zero Trust Network Access (ZTNA)** principles with granular device controls.

## Monitoring, Reporting & Alerts

**Device compliance dashboards**.

**Real-time reporting** for configuration drift, app deployment, policy errors.

Integration with **Log Analytics / Azure Monitor** and **Microsoft Sentinel**.

Alerts for jailbreak/root detection, compromised devices, non-compliance.

## Hybrid Integration with On-Prem Infrastructure

**Co-management** with Configuration Manager (SCCM).

**Hybrid Azure AD Join support** for GPO + Intune coexistence.

Support for **on-prem certificate deployment** via **Intune Connector** for SCEP/NDES.

On-prem app deployment and Win32 app hosting via line-of-business strategies.

## Automation and Governance

**Role-based access control (RBAC)** within Intune and MEM.

**Scripted deployments** using Graph API / PowerShell SDK.

**Policy as Code** via **Azure DevOps pipelines**, GitHub Actions, or Terraform for Intune.

**Audit logs** and **compliance policies versioning**.

## Appendix:

Microsoft Links

[CyberAutomationX](https://github.com/CyberAutomationX)/[SecureAzCloud-Scripts](https://github.com/CyberAutomationX/SecureAzCloud-Scripts)

## Technical Deep Dive for Network/Infra Architects

<details>
<summary> <strong> ### Device Lifecycle Management </strong></summary>
<br>
*Platform Scope: Windows 10/11, macOS, iOS/iPadOS, Android Enterprise*

#### 1. Provisioning & Enrollment

**Windows 10/11 (Autopilot + AAD Join/Hybrid AAD Join)**

- **Windows Autopilot** provisions devices with:
  - **Pre-defined deployment profiles** (user-driven or self-deploying)
  - Azure AD Join or Hybrid Azure AD Join
  - Optional **ESP (Enrollment Status Page)** to block access until provisioning is complete

- **Zero Touch Provisioning (ZTP):**
  - Devices are added to Autopilot via **hardware hashes** or OEM direct integration
  - Intune auto-applies policy, apps, configurations

- **Group Tagging**: Enables role-/location-based assignment via dynamic groups

**macOS (Apple Automated Device Enrollment via ABM)**

- Device is enrolled using **ADE (formerly DEP)** through Apple Business Manager
- Combined with **Intune MDM enrollment profile** for supervised mode
- Paired with **User Affinity** or **Shared iPad mode** based on use case
- ZTP allows devices to be **drop-shipped** and managed on first boot

**iOS/iPadOS**

- ADE + Supervised Mode (over-the-air, no need for Apple Configurator)
- Useful for:
  - Kiosk mode
  - Single App Mode (POS, retail terminals)
  - BYOD scenarios (via **App Protection Policies**, no full device enrollment)

**Android Enterprise**

- **Zero Touch Enrollment (ZTE)** for corporate-owned devices
- **Work Profile (BYOD)** and **Dedicated/Kiosk Mode** supported
- OEM enrollment tokens or QR-code based provisioning
- **Fully Managed** or **COPE (Corp-Owned, Personally Enabled)** depending on policy

#### 2. Device Compliance Enforcement

Compliance policies are central to your **Zero Trust conditional access** posture, as outlined in your reference architecture. They drive enforcement and segmentation.

**Key Policy Elements:**

- **Health Attestation**: Check Windows Defender ATP status
- **Encryption**: BitLocker for Windows, FileVault for macOS, native encryption on iOS/Android
- **OS Version Enforcement**: Block outdated, vulnerable versions
- **Jailbroken / Rooted Detection**
- **Threat Risk Level Integration**: Defender for Endpoint or 3rd-party AV integration

**Example:**

Policy: Block access unless device is compliant AND Defender Risk Level = Low

Effect: User is denied access to M365 if running outdated macOS, jailbroken iPhone, or unmanaged Android

#### 3. Remote Actions & Operational Management

Network and Infrastructure Architects must understand how **remote actions** support Day 2 operations, incident response, and service desk offload.

| **Action** | **Supported OS** | **Use Case** |
|------------|------------------|--------------|
| **Wipe** | All | Decommission, lost/stolen devices |
| **Retire** | All | BYOD cleanup, remove corporate data only |
| **Restart** | Windows 10/11 | Remote support, patching resets |
| **Lock** | Windows/macOS/iOS | Security response to suspected compromise |
| **Sync** | All | Force policy refresh (e.g., after CA changes) |
| **Locate** | iOS/macOS (supervised) | Lost device tracking |

These actions integrate with **Help Desk workflows**, and in your architecture, could be triggered via **ITSM tools like ServiceNow** using Graph API integrations or automation playbooks (e.g., Logic Apps or Power Automate).

#### 4. Application of Lifecycle Stages

| **Stage** | **Intune Capability** | **Example** |
|-----------|------------------------|-------------|
| **Procurement** | OEM registration, Autopilot import | Dell direct-to-user shipment with Autopilot |
| **Provisioning** | Enrollment profile assignment, ESP | ZTP for kiosk iPads via ADE |
| **Operational** | Compliance, app updates, patching | Monthly Windows Update Rings |
| **Support** | Remote actions, diagnostics, shell access | Remote lock + retire via Intune portal |
| **Decommission** | Wipe/Retire, license recycle | Wipe ex-employee MacBook with audit logging |

#### 5. Architecture Alignment with Your Blog Posts

| **Reference Element** | **Lifecycle Tie-In** |
|------------------------|----------------------|
| **"User to device to service" security plane (EUC blog)** | Intune defines **device trust**, controls the bridge from user to service |
| **Conditional Access integration** | Enforces lifecycle states (e.g., block stale devices) |
| **Intelligent automation** (Endpoint Ref Arch) | Auto-remediation workflows with Intune + Defender |
| **Support for frontline, task-based users** | Shared device provisioning (e.g., iPadOS Shared iPad mode, Windows Kiosk Mode) |

**Bonus: Zero Trust Lifecycle Enforcement**

Using Conditional Access + Device Compliance:

- **Example Flow**: Device out of compliance → Intune flags → CA blocks M365 access → Notification sent via Endpoint Manager → Auto-remediation script triggered → Device re-evaluated

- **Useful Tools**:
  - **Intune Remediations** (Proactive Scripts + Detection Rules)
  - **Defender Security Baselines**
  - **Graph API** for automation and compliance analytics
</details>
### Application Management in Microsoft Intune

**Scope**: Covers both **Mobile Application Management (MAM)** and **Mobile Device Management (MDM)**-based app delivery across:

- **Windows 10/11 (Win32, MSI, UWP)**
- **macOS**
- **iOS/iPadOS**
- **Android (Enterprise & BYOD)**

#### 1. Intune App Protection Policies (MAM Without Enrollment)

**Use Case: BYOD / Personal Devices**

- No full device enrollment or corporate wipe
- Controls **corporate data within apps only**

**Core Capabilities**

| **Control Type** | **Description** |
|------------------|-----------------|
| **Selective Wipe** | Wipe corporate data from apps (not device) |
| **Copy/Paste Restrictions** | Block between managed and unmanaged apps |
| **Save-As Restrictions** | Block saving to local or personal storage |
| **Screen Capture** | Prevent screenshots on Android/iOS |
| **Data Transfer** | Block data sharing to non-managed apps |
| **Conditional Access** | Require policy-compliant apps to access services |

**Supported Apps**

- Microsoft 365: Outlook, Teams, Edge, OneDrive
- **Line-of-Business Apps**: Wrapped with Intune SDK or via App Wrapping Tool
- Third-party apps in **Intune MAM SDK ecosystem**

**Platform Notes**

| **Platform** | **Notes** |
|--------------|-----------|
| **iOS/Android** | Full MAM-WE support via public app stores |
| **macOS/Windows** | Primarily MDM-based; MAM is limited to Edge/Outlook (via Conditional Access) |

✳ **Integration Tip from your blog**: MAM-WE maps well to **"Conditional Workspace"** patterns — e.g., users access corporate data through protected apps only, without a full device trust requirement.

#### 2. Application Deployment Scenarios

**A. Windows Apps**

| **Type** | **Examples** | **Deployment Notes** |
|----------|--------------|----------------------|
| **Win32** | .exe, .ps1 | Most flexible. Use Win32 App Packaging Tool. |
| **MSI** | Office, Adobe | Native support with detection rules |
| **UWP** | Store apps | Direct from Microsoft Store integration |
| **Microsoft Store for Business (MSfB)** | Legacy model, being deprecated (migrate to Winget) | |

**B. macOS Apps**

- .pkg deployments
- Post-install scripts for configuration
- No store integration — requires sideloading apps or using Apple Business Manager + VPP

**C. iOS/iPadOS & Android Apps**

| **Type** | **iOS/iPadOS** | **Android** |
|----------|----------------|-------------|
| **Store Apps** | Via Apple VPP | Managed Google Play |
| **LOB Apps** | IPA via ABM | APK via Managed Play Store |
| **Web Apps** | Web shortcuts with browser restrictions | Same |
| **Conditional Launch Controls** | Jailbreak/root check, app version enforcement | Similar across platforms |

These app types align well with your concept of **persona-driven app sets** (frontline vs knowledge workers).

#### 3. App Protection Policy Enforcement Examples

| **Scenario** | **Policy** | **Effect** |
|--------------|--------------|------------|
| **BYOD access to Outlook** | MAM-WE with PIN + Save-as block | User can access Outlook, but cannot copy data to Notes app or save attachments to personal cloud |
| **Managed Android kiosk** | App protection + device lock-down | Access limited to one managed app in single-app mode |
| **Shared device in logistics** | Per-app sign-in with SSO + MAM | One device, multiple users, app policy controls per session |

#### 4. App VPN, Per-App Networking & Data Leak Protection

**App-based VPN (iOS/Android)**

- Integrates with **Zscaler, Cisco AnyConnect, Palo Alto GlobalProtect**
- Enforces VPN only for corporate apps (not whole device)
- Reduces attack surface on unmanaged devices

**DLP Integration**

- Intune App Protection enforces:
  - No sharing to unmanaged apps
  - No cloud sync outside approved services (e.g., OneDrive for Business)
- App-based conditional launch (e.g., block if rooted, outdated OS, etc.)

#### 5. Integration with Endpoint Protection and CA

- Conditional Access enforces:
  - App version compliance
  - App protection policy presence
  - Intune app management state
- Integration with **Microsoft Defender for Endpoint**:
  - High-risk apps/devices can be blocked automatically
  - Use **App Control Policies** to restrict executables

This maps directly to your **"identity, app, device" trust plane model** in the EUC Reference Architecture.

#### 6. App Lifecycle Flow

| **Lifecycle Stage** | **Intune Capability** |
|---------------------|------------------------|
| **Procurement** | Microsoft Store, Apple VPP, Managed Play |
| **Deployment** | Targeted groups, dynamic assignment |
| **Configuration** | App config policies (Outlook mail settings, VPN profiles) |
| **Protection** | App Protection Policies, Conditional Access |
| **Update** | App version enforcement, detection rules |
| **Retirement** | App removal or selective wipe for BYOD |

#### 7. Architectural Alignment with Your Blog Posts

| **Architecture Concept** | **MAM/App Management Tie-In** |
|--------------------------|------------------------------|
| **Modern App Delivery** | Store-based, wrapped apps, Win32, zero trust-aware |
| **Persona-Based App Delivery** | Role-based app assignments via dynamic groups |
| **Cloud-Native Security** | MAM policies + Conditional Access + Defender Risk |
| **Hybrid Workforce Enablement** | MAM-WE protects data on unmanaged devices |
| **ZTP + App Delivery** | Autopilot integrates apps at provisioning for corporate users |

### Endpoint Security & Zero Trust Integration

#### 1. 🔄 Microsoft Intune & Cisco ISE NAC Integration (Compliance API)

This integration allows **Cisco ISE** to query Intune for **real-time device compliance status** before granting network access (802.1X/EAP-TLS).

**✅ Key Integration Components:**

| **Component** | **Description** |
|--------------|------------------|
| **Intune Compliance Policies** | Define criteria: encryption, AV status, OS version, device not jailbroken/rooted. |
| **Azure AD Conditional Access** | Enforces resource access based on compliance (device must be marked compliant). |
| **Microsoft Graph Security API (Compliance API)** | Cisco ISE queries device compliance via this API. |
| **Cisco pxGrid** | Acts as a broker between ISE and Microsoft Intune. |

**⚙️ Integration Flow:**

1. User connects to Wi-Fi/Ethernet (802.1X auth).
2. Cisco ISE identifies user/device.
3. ISE uses pxGrid to request **device compliance** status from Intune via the Compliance API.
4. If **compliant**, ISE allows access (VLAN permit); if **non-compliant**, redirects to remediation VLAN or captive portal.
5. Optionally enforces **dynamic ACLs** or **SGTs (Scalable Group Tags)**.

**🛠 Design Considerations:**

- Requires **Azure AD P1+ licensing**.
- Device must be **Azure AD-joined or hybrid-joined** and enrolled with Intune.
- Must configure **Trusted IPs** in Conditional Access to prevent looping issues with hybrid auth scenarios.
- Works best with **certificate-based authentication** (e.g., EAP-TLS via Intune + SCEP).

#### 2. 🛡️ Endpoint Security Controls via Intune

Network architects should understand **Intune's policy enforcement at the endpoint**, especially as it affects posture assessment.

**🔑 Endpoint Security Policy Types:**

- **Antivirus (Microsoft Defender for Endpoint)**
- **Disk Encryption (BitLocker / FileVault)**
- **Firewall (Windows Defender Firewall profiles and rules)**
- **Attack Surface Reduction (ASR)**
- **Security Baselines** (NIST/CIS-aligned templates)
- **Credential Guard / Application Guard** (for protected sessions)

These controls ensure that endpoints are hardened **before they can participate in the network**, enforcing Zero Trust assumptions.

#### 3. 🌐 Support for Zero Trust Network Access (ZTNA) Principles

**🔍 Core ZTNA Capabilities Enabled by Intune:**

| **Principle** | **Intune Capability** |
|---------------|------------------------|
| **Verify explicitly** | Identity + device posture via Intune + Azure AD CA. |
| **Use least-privilege access** | Scoped Conditional Access, device tagging, user roles. |
| **Assume breach** | Continual enforcement of compliance, AV signals from Defender, app control. |

**🔧 Device Controls in ZTNA:**

- Device must be **compliant**, **health-checked**, and **identity-bound**.
- Conditional Access can enforce:
  - Device **must be Intune-enrolled**.
  - Device must meet **Defender risk score thresholds**.
  - Access **only from specific network locations (trusted IPs)**.
  - Only allow **managed apps** on **unmanaged devices**.
- **Session controls** using Microsoft Defender + Defender for Cloud Apps (MCAS):
  - Block upload/download based on sensitivity or device posture.

**📡 Network Relevance:**

- Combined with **per-app VPN** or **Zscaler/Netskope**, you can restrict traffic to only secured tunnels.
- Can trigger **remediation flows** or quarantine VLANs via NAC if ZT posture is violated.

**🧩 Architectural Integration Snapshot**

Here's a **layered view** of how these elements integrate:

```
+----------------------------------------------------------+
| Microsoft Intune                                         |
| - Enroll Devices  - Apply Compliance Policies            |
| - Configure VPN/Wi-Fi  - Push Security Baselines         |
+----------------------------------------------------------+
                          ⇅ Microsoft Graph Compliance API
+----------------------------------------------------------+
| Cisco ISE (NAC Layer)                                    |
| - 802.1X Auth  - pxGrid for posture integration          |
| - VLAN / ACL Enforcement                                 |
+----------------------------------------------------------+
                          ⇅ SAML / OIDC + Conditional Access
+----------------------------------------------------------+
| Azure AD / Entra ID                                      |
| - Conditional Access (compliance + risk-based)           |
| - MFA / App Control / Device Risk                        |
+----------------------------------------------------------+
                          ⇅ Defender for Endpoint (Signal Sharing)
+----------------------------------------------------------+
| Defender for Endpoint / Microsoft Sentinel               |
| - Threat protection  - Risk-based session policies       |
| - Integration with MCAS (CASB) and ZTNA enforcement      |
+----------------------------------------------------------+
```

**🧭 Summary — What a Network & Infra Architect Should Know**

| **Domain** | **Key Knowledge** |
|------------|-------------------|
| **NAC Integration** | How Cisco ISE queries Intune via pxGrid and Compliance API. |
| **Policy Enforcement** | Designing network access policies based on compliance status (802.1X + Conditional Access). |
| **ZTNA** | Applying device identity and health for access control, combined with network segmentation. |
| **VPN/App Isolation** | How Intune per-app VPN + managed apps enforce network/data boundaries. |
| **Network Signal Flow** | The sequence between device connect, ISE posture check, Intune status, and conditional access response. |

### Monitoring, Reporting & Alerts in Modern EUC Architecture

**🎯 Purpose & Value**

The **"Monitoring, Reporting & Alerts"** capability ensures:

- Continuous **compliance posture awareness**
- Rapid detection of configuration drift and security anomalies
- Integration with **SIEM/SOAR** pipelines for automated response
- Confidence in **policy enforcement, app delivery, and user experience**

This aligns directly with your reference architecture's **EUC Operations**, **Zero Trust enforcement**, and **device trust pillar** by enabling **automated observability**.

**🧱 Key Capability Breakdown**

#### 1. ✅ Device Compliance Dashboards

- **Tooling**: Intune Compliance Center, Microsoft Endpoint Manager, Entra ID Portal
- **Function**: Aggregates compliance state based on:
  - **Platform-specific settings** (e.g., BitLocker, AV status, firewall, OS version)
  - **Custom compliance scripts** (e.g., checking for specific reg keys or apps)
- **Use Cases**:
  - Show non-compliant devices per platform/user group
  - Feed compliance signals to **Conditional Access** (CA) for Zero Trust enforcement
- **Best Practices**:
  - Define a **"baseline" compliance policy** for all device classes
  - Integrate with **RBAC + scope tags** to enable fleet-specific reporting (e.g., BYOD, kiosk, hybrid user)

#### 2. 📡 Real-Time Reporting for Configuration Drift, App Deployment & Policy Errors

- **Sources**:
  - Intune's built-in "**Endpoint analytics**"
  - **Proactive Remediation Scripts** (PowerShell)
  - **Update Compliance** (for patch levels)
  - **App install status** via Intune Win32 logs
- **Drift Detection**:
  - **Baselines vs current state** (e.g., Defender AV turned off, BitLocker disabled)
  - Policy assignment vs actual application (e.g., configuration profiles not applying)
- **Modern Options**:
  - Export logs to **Log Analytics workspace**
  - Use **custom KQL queries** to flag out-of-policy endpoints
- **Example Queries**:

```
IntuneDevices
| where ComplianceState != 'Compliant'
| summarize count() by DeviceOSType, DeviceName
```

#### 3. 🧩 Integration with Log Analytics, Azure Monitor & Microsoft Sentinel

- **Objective**: Enterprise-grade **observability**, threat correlation, alerting
- **Capabilities**:
  - Route Intune diagnostic logs to **Azure Log Analytics**
  - Enrich with **Defender for Endpoint** signals (e.g., vulnerability, AV)
  - **Workbooks**: Custom dashboards combining:
    - Compliance trends
    - App deployment success/failure
    - Endpoint health scores
  - **Microsoft Sentinel**:
    - Use **Data Connector** for Intune
    - Create **Analytics Rules** (e.g., non-compliant device + suspicious login = high-priority alert)
- **Outcome**: Unified endpoint security + ops view within **SOC and platform engineering teams**

#### 4. 🚨 Alerts for Jailbreak / Root Detection, Compromised Devices, Non-Compliance

- **Platforms**:
  - iOS/Android: Jailbreak/root detected via **Intune device compliance policies**
  - Windows/macOS: Leverage **Defender ATP** & **Compliance Scripts**
- **Alert Methods**:
  - **Intune built-in alerting** (for policy deployment, compliance failures)
  - **Microsoft Sentinel alerts** via KQL + rule sets
  - **Logic Apps** for:
    - Slack/Teams notifications
    - Auto-remediation (e.g., remove access, isolate device, ticket in ServiceNow)
- **Example: Sentinel Alert Rule**:

```
DeviceComplianceOrg
| where ComplianceState == 'NonCompliant'
| join kind=inner (
  SecurityAlert | where AlertName contains "Suspicious Sign-in"
) on DeviceId
```
→ Can trigger:
- **Email to SOC**
- **Autonomous CA policy block**
- **Intune remediation script**

**🧩 Integration Patterns with Your EUC Reference Architecture**

| **EUC Tier** | **Monitoring Touchpoint** |
|--------------|---------------------------|
| **Identity & Access** | Conditional Access + Compliance signal ingestion |
| **Device Management (MDM)** | Intune + Defender data streams → Log Analytics → Sentinel |
| **App Delivery** | Intune App telemetry + Win32 logs for success/failure analysis |
| **Endpoint Protection (AV, EDR)** | Defender alerts + Sentinel correlation |
| **Observability & Automation** | Logic Apps, Azure Monitor alerts, ServiceNow integration |

**🌐 Real-World Implementation Blueprint**

**1. Data Collection Flow**

Intune ➜ Log Analytics ➜ Azure Monitor ➜ Sentinel ➜ Alert/Automation  
↳ Workbooks & Dashboards

**2. Alerting & Action Flow**

Sentinel Rule ➜ Logic App ➜ Notify | Auto-Remediate | CA Block | Raise Incident

**3. Dashboards to Include**

- Non-Compliant Devices Over Time
- Top Policy Errors (e.g., profile failed to apply)
- Devices Without Required App X
- Jailbroken / Rooted Mobile Devices

**🧭 Maturity Model**

| **Level** | **Capability** |
|-----------|----------------|
| Basic | Manual review of Intune compliance reports |
| Intermediate | Scheduled workbook reports + alerts in Monitor |
| Advanced | Sentinel-integrated alerts + automated response |
| Optimized | Predictive analytics & AI-based risk scoring |

### Hybrid Integration with On-Prem Infrastructure (Intune + SCCM + AD)

**Objective**: Enable modern management **without breaking existing enterprise investments** in Group Policy, Configuration Manager, and on-prem app delivery — while **bridging to Zero Trust** and cloud-first endpoint strategies.

#### 🔁 1. Co-management with Microsoft Configuration Manager (SCCM)

**🔹 Purpose**

Allows devices to be **managed concurrently** by both **Intune (cloud)** and **SCCM (on-prem)**, enabling **phased workloads migration**.

**🔹 Architecture**

- Devices are joined to **Active Directory (AD)** and **enrolled into Intune**
- Co-management agent (SCCM client) connects to on-prem infrastructure
- Device syncs with **Azure AD** and **Microsoft Endpoint Manager**

**🔹 Workload Split Options**

You can configure which workloads are managed by SCCM vs. Intune:

| **Workload** | **Managed by Intune?** |
|--------------|-------------------------|
| Compliance Policies | ✅ |
| Device Configuration | ✅ |
| Windows Updates | ✅ |
| Endpoint Protection | ✅ |
| App Deployment | Gradual shift from SCCM to Intune |
| Resource Access (Wi-Fi/VPN) | ✅ |

Your blog's principle of **progressive modernization** aligns well here: allow legacy infra to coexist while progressively migrating to cloud-native control.

#### 🧩 2. Hybrid Azure AD Join (HAADJ) for GPO + Intune Coexistence

**🔹 Use Case**

- Enables **domain-joined** devices to **register in Azure AD**
- Supports **Intune enrollment + GPO** coexistence

**🔹 Why It Matters**

- Enterprises with heavy Group Policy Object (GPO) dependencies can maintain those controls **while layering Intune MDM policies**
- Enables **Conditional Access**, **Cloud-based SSO**, and **Endpoint Analytics**

**🔹 Architecture Notes**

- Requires **Azure AD Connect**
- Devices authenticate via **Kerberos to on-prem AD**, while still leveraging **cloud identity for modern apps**

**🔹 GPO vs Intune Policy Handling**

| **Scenario** | **GPO** | **Intune** |
|--------------|---------|------------|
| Legacy App Config | ✅ | ❌ |
| Modern Security Baselines | ❌ | ✅ |
| Overlapping Policies | GPO wins if conflict | |

From your EUC architecture lens: HAADJ is critical for **stage-2 modernization** — still tethered to AD, but enabling cloud app access and identity controls.

#### 🔐 3. On-Prem Certificate Deployment via Intune SCEP/NDES Connector

**🔹 Purpose**

Provide certificates for:

- Wi-Fi (802.1X)
- VPN (e.g., Cisco AnyConnect, Palo Alto)
- App authentication (e.g., Outlook with S/MIME)

**🔹 Components**

| **Component** | **Role** |
|---------------|----------|
| Intune Connector | Relays certificate requests securely from Intune to on-prem |
| NDES | Issues SCEP certificates via Microsoft CA |
| Microsoft CA | Generates and tracks issued certs |

**🔹 How It Works**

1. Intune sends certificate request to Connector
2. Connector relays to **on-prem NDES server**
3. Certificate is issued via SCEP
4. Intune installs cert on device

**🔹 Security Best Practices**

- Use **HTTPS-only communication**
- Place NDES/Connector behind firewall + secure ACLs
- Log and monitor issued certs via CA

Critical to enabling **Zero Trust Wi-Fi onboarding** (as noted in your blogs), especially for **macOS/iOS and Android** where cert-based auth is preferred.

#### 🗃️ 4. On-Prem App Deployment & Win32 Hosting Strategies

**🔹 Hybrid App Delivery Needs**

For legacy or large Win32 apps not yet suitable for full cloud delivery:

- App content may be **hosted on-prem**
- Distribution still managed via **Intune or SCCM**

**🔹 Key Options**

**A. Intune Win32 App Hosting
## Application Deployment

- Package app as .intunewin file
- Upload to **Intune Storage**
- Works great for:
  - Line-of-business (LOB) apps
  - Office add-ins
  - Win32 installation scripts

**Bandwidth-optimized with Delivery Optimization + Peer Caching**.

### B. Cloud Management Gateway (CMG) for SCCM

- Allows SCCM clients to **download content from cloud endpoints**
- Bridges internet-based clients to on-prem hosted apps

### C. On-Prem Hosted Installers with Intune Script Deployment

- Intune delivers **PowerShell scripts** that pull from internal file shares or DFS paths
- Requires:
  - **VPN** or **ExpressRoute/Site-to-Site**
  - Internal name resolution (DNS)
  - Robust error handling

## Content Delivery Patterns

| **Pattern** | **Example** |
|-------------|-------------|
| Intune Hosted | Win32 app in .intunewin |
| Hybrid (Intune + DFS) | Script points to \\\\corp\\share\\installer.exe |
| SCCM App Model | Use deployment types, detection rules, pre-reqs |

## Aligning to Your Reference Architecture

| **Hybrid Component** | **EUC Architecture Implication** |
|---------------------|----------------------------------|
| Co-management | Supports transitional state for existing SCCM-managed fleets |
| HAADJ | Enables Zero Trust access control without AD decommissioning |
| SCEP Connector | Bridges modern management to on-prem PKI and Wi-Fi/VPN auth |
| Hybrid App Delivery | Retains access to legacy/internal apps as part of app rationalization |

This supports your **Hybrid Workforce** model — balancing modern endpoints with grounded support for **legacy infra** and **LOB applications**.

## Summary: Why This Matters for Network & Infra Architects

| **Priority** | **Capability** | **Why It's Important** |
|--------------|----------------|------------------------|
| ✅ | Co-management | Maintain operational continuity during cloud migration |
| ✅ | HAADJ | Preserve AD trust while enabling modern identity |
| ✅ | SCEP/NAC | Cert-based Wi-Fi/VPN access for ZTNA |
| ✅ | On-Prem App Delivery | Support bandwidth-heavy or compliance-bound apps |
| ✅ | Policy Reconciliation | Align legacy GPO with Intune baselines over time |

**Group Policy (GPO) vs Intune** and **SCCM vs Intune App Delivery** — based on **device persona**, **use case**, and **modernization maturity**.

## GPO vs Intune MDM Policy -- Technical Decision Matrix

| **Criteria** | **Use GPO 🏢 Legacy/Hybrid** | **Use Intune MDM Policies ☁️ Cloud-First/Modern** |
|-------------|--------------------------|--------------------------------------|
| **Device Join Type** | On-prem AD (with HAADJ) | AADJ / AADJ + Entra ID P2 |
| **Network Connectivity** | Corporate LAN/VPN | Internet-based / Hybrid Workforce |
| **Management Stack** | SCCM/Group Policy | Intune MDM/Autopilot |
| **Policy Scope** | Computer/User | Device/User (AAD-based targeting) |
| **Policy Granularity** | Deep Windows internals | Focused on security and modern management controls |
| **Policy Change Propagation** | Fast (on boot/logon) | Delayed/periodic via MDM sync |
| **Policy Examples** | Registry edits, drive maps | BitLocker, Defender AV, Firewall, Password rules |
| **Zero Trust Readiness** | ❌ No native CA support | ✅ Supports Conditional Access, Compliance |
| **Platform Support** | Windows-only | Windows, macOS, iOS, Android |
| **Decommissioning Roadmap** | Retiring over time | Strategic endpoint control plane |
| **Recommended For** | Static desktop fleet, labs | BYOD, field staff, hybrid/remote workers |

## SCCM vs Intune App Deployment -- Technical Decision Matrix

| **Criteria** | **Use SCCM 🏢 Legacy** | **Use Intune ☁️ Modern** |
|--------------|---------------------|-------------------------|
| **Device Join Type** | AD + SCCM client | AADJ / HAADJ + Intune |
| **App Size/Type** | >2GB installers, legacy LOB | Modern apps, .intunewin, UWP, iOS/Android |
| **Deployment Location** | On-prem DFS/SCCM DPs | Intune CDN (Azure Blob) |
| **Network Constraints** | LAN-optimized, P2P | Internet-first (use Delivery Optimization) |
| **Install Logic/Dependencies** | Complex installers, App-V | Win32 apps with detection scripts |
| **Targeting Granularity** | AD Security Groups, OU | AAD Groups, Dynamic Targeting |
| **Self-Service Portal** | Software Center | Company Portal (cross-platform) |
| **Co-management Consideration** | Keep SCCM workload | Migrate to Intune workload |
| **Content Delivery Optimization** | BranchCache / P2P | Delivery Optimization |
| **Recommended For** | Call centers, LOB desktops | Hybrid workers, mobile staff, kiosks |

## Use Case Persona-Based Guidance

| **Persona** | **Policy Management** | **App Delivery** | **Notes** |
|-------------|----------------------|------------------|-----------|
| **Back Office Staff (HQ)** | GPO + Intune hybrid | SCCM primary | Stable network, legacy support needed |
| **Field Worker (Mobile)** | Intune only | Intune (Win32, MAM) | Uses LTE/Wi-Fi; no corp LAN |
| **Contractor / BYOD** | MAM (no enrollment) | MAM + Conditional Access | No device enrollment |
| **Factory / Kiosk Device** | Intune + Autopilot | Intune + offline installer | Shared login, locked down |
| **Executive (Hybrid Work)** | Intune + Defender Policies | Intune + Win32/Company Portal | ZTNA enforcement, sensitive apps |

## Decision Criteria Tree

→ Is the device enrolled into Intune?

→ YES:
  → Is the device AD-joined (HAADJ)?
    → YES → Use Co-Management: Decide per workload (e.g., compliance = Intune; app = SCCM)
    → NO → Use full Intune (policy + apps)

→ NO:
  → Is the device BYOD?
    → YES → Use MAM + Conditional Access (no device control)
    → NO → Manage via GPO + SCCM (legacy path)

## Supporting Modernization Strategy

Use this matrix to:

- Gradually **move workloads to Intune** (start with compliance, then apps)
- **Replace GPO** where supported by MDM policies (track gaps)
- Use **Device Scope Tags and Role-Based Access Control (RBAC)** to support separation of duties across hybrid teams
- Monitor success with **Endpoint Analytics**, **Update Compliance**, and **Security Baselines**

# Conditional Access & Network Enforcement -- Deep Dive

This document provides a consolidated deep dive on **Conditional Access (CA)** and **Network Enforcement**, combining identity, device, and network signals to support **Zero Trust** principles—**never trust, always verify**. It integrates **Microsoft Entra ID (formerly Azure AD)**, **Intune**, **Microsoft Defender for Endpoint**, **VPN/Wi-Fi**, and **Network Access Control (NAC)**. This is aligned with the architecture and intent outlined in the Enterprise EUC Reference Architecture.

## Strategic Purpose in Zero Trust EUC

Conditional Access is the **policy engine** for EUC that makes dynamic access decisions based on:

- Identity posture (user/group membership, risk level)
- Device posture (compliance, join status, OS/platform)
- Network context (trusted IPs, VPN/NAC, geolocation)
- Real-time risk signals (from Defender or Entra ID Protection)
- Application sensitivity (SaaS, internal, or critical workloads)

The **core enforcement point (PEP)** is **Entra ID**, with conditional logic tightly coupled with **Intune**, **Defender for Endpoint**, and **network-based controls** to drive secure access to apps and data.

## Core Capabilities & Enforcement Layers

### 1. Policy-Based Access Control

**What**: Centrally managed CA policies that combine identity, device, app, location, and risk signals.

**How**: Defined using conditional logic:

*If* user is in Group X, accessing App Y, from unmanaged device → *Then* require MFA + compliant device.

Use Cases:

- Enforce compliant & hybrid AAD-joined devices for Office 365, Salesforce, ServiceNow.
- Block access from unsupported OS platforms or high-risk sign-ins.
- Bypass MFA from trusted IP ranges or VPN.

Best Practices:

- Leverage **Named Locations** for geolocation- or IP-based rules.
- Use **Entra ID Protection** for sign-in and user risk scoring.

### 2. Device Compliance & Trust (Intune + Entra)

**Device Requirements**:

- Must be marked **compliant** (via Intune) or **Hybrid Azure AD Joined**.
- Use **Device Filters** for fine-grained access (e.g., only iOS 16+ managed devices).

**Example**: Block non-compliant BYO Windows/macOS/mobile devices from accessing sensitive workloads.

### 3. Defender for Endpoint Risk-Based Access

**Why**: Integrate **real-time device risk** posture into CA decisions.

**Risk Enforcement Flow**:

1. Defender detects device anomalies (AV status, vulnerabilities).
2. Risk level (Low/Medium/High) sent to Entra ID.
3. CA evaluates: High-risk → Block, Medium → Require MFA, Low → Allow.

**Use Case**: Prevent access from compromised or vulnerable endpoints across any platform.

### 4. Network-Aware Conditional Access

**Techniques**:

- Use **Named Locations**: Define trusted IPs (corporate offices, VPN gateways).
- Configure **Conditional Access Filters**: OS type, client app, device platform.
- Combine **NAC posture** + **network profile enforcement** to evaluate device trust before granting access.

**Example Policies**:

- Finance apps → accessible only from corporate Wi-Fi or trusted VPN.
- Skip MFA from office IPs with validated NAC posture.

### 5. Wi-Fi & VPN Enforcement via Intune

**Wi-Fi Profiles**:

- Push **802.1X** profiles using certificates (SCEP/PKCS).
- Secure SSIDs with EAP-TLS or PEAP.

**VPN Profiles**:

- Define **Per-App VPN** for iOS/Android/Windows.
- Integrate with 3rd-party VPN providers: Zscaler, Palo Alto, Cisco AnyConnect, F5 APM.

**Policy**: Block access if VPN/Wi-Fi profiles are missing, misconfigured, or expired.

## Mapping to EUC Architecture Pillars

| **Pillar** | **Role of CA & Enforcement** |
|------------|------------------------------|
| **Identity Trust** | Entra ID + Conditional Access + Entra ID Protection |
| **Device Trust** | Intune Compliance + Defender Risk + Device Filters |
| **App Delivery** | App access governed by per-platform CA policies |
| **Data Protection** | Trusted Wi-Fi, Per-App VPN, Network Restrictions |
| **Zero Trust** | Runtime enforcement—identity, device, and network |

## Architecture Workflow Example

**Conditional Access Evaluation**:

1. User requests app access (e.g., Microsoft 365).
2. Entra ID triggers evaluation:
   - Is user/group risky?
   - Is device compliant + hybrid-joined?
   - Trusted network IP/VPN/NAC?
   - Device risk from Defender?
3. Access Control Decision:
   - ✅ Allow with MFA
   - 🚫 Block access
   - ⚠️ Require compliant device or VPN

## Real-World Conditional Access Scenarios

| **Scenario** | **Policy Logic** |
|--------------|------------------|
| Block risky mobile access | Device risk = High AND platform = iOS/Android → Block access |
| Require compliant device for SharePoint | App = SharePoint AND device NOT compliant → Block access |
| BYO access (Outlook + MAM only) | Platform = iOS AND unmanaged → Allow Outlook only + App Protection Policy |
| Bypass MFA in trusted office | IP = Named Location (office/VPN) → Skip MFA |
| Finance app via VPN only | Require device on trusted IP + Per-App VPN profile |

## Maturity Model

| **Level** | **Capability Description** |
|-----------|----------------------------|
| 1️⃣ Basic | Manual MFA enforcement; no device or risk evaluation |
| 2️⃣ Intermediate | Conditional Access with device compliance and named locations |
| 3️⃣ Advanced | Integrated Defender risk + platform-specific app control |
| 4️⃣ Optimized | Full runtime policy enforcement with automation (Sentinel, Intune, etc.) |

## Reporting, Simulation & Policy Analytics

- **Entra Sign-In Logs**: See each login's CA policy result.
- **Azure Monitor & Sentinel**: Correlate CA failures or non-compliant devices.
- **Policy Analytics (Preview)**: Simulate and test policy impact before rollout.
- **Workbooks**: Dashboards for blocked attempts, high-risk users, and app-specific access patterns.

## Admin Tools & Portals

| **Tool** | **Functionality** |
|----------|-------------------|
| Entra Admin Center | Define, simulate, and analyze Conditional Access policies |
| Intune Admin Center | Deploy VPN/Wi-Fi profiles, enforce compliance policies |
| Defender Security Portal | View device health and risk posture |
| Azure Monitor / Sentinel | Monitor CA outcomes, trigger alerts on violations |

**Microsoft links:**

[CyberAutomationX](https://github.com/CyberAutomationX)/[SecureAzCloud-Scripts](https://github.com/CyberAutomationX/SecureAzCloud-Scripts)


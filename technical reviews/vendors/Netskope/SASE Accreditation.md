
# SASE Accrediation

## SSE < >

### Origin of Security Service Edge
- SWG:   secure web traffic, scan for malware & enforce sec polices.
- CASB:  secure apps and data, visibility and control of cloud usage.
- VPN:   secure remote access to corporate resources.

### SSE Functions
- *Web & Cloud Apps*           visibility and control
- *Remote Working*             modernize
- *Threat & risky Activity*    detect and mitigate
- *Identify & Protect*         sensitive data & information

### Zero Trust Policy in Practice
- Reduce attack surface
- Focus on the protect surface
- Risk Mitigation: from Device/User to Action
- All Destinations

ZT Policy Layers
- Device/User
- Location
- App Risk
- Instance
- Activity
- Behavior
- Data
- Action

| **Function** | **Components** |
|--------------|------------------|
| Access Control:| Device/User, Location, App Risk, Instance, Activity|
| Data Protection & DLP:| Behavior, Data, Action|
| Action:| Allow, Warn, Coach, Block, Prompt for MFA|

### Evolution of SSE
- from Web Proxy to App Proxy
  
Web, 10% | SaaS 85% | IaaS 5%

Inline Cloud Proxy

### SSE Modes: Visibility into Cloud & Web

| **Mode** | **Function** |
|--------------|------------------|
| Forward proxy mode:| endpoint agents or VPN clients|
| Reverse Proxy mode:| identity services, use-cases: BYOD or Unmanaged devices, external auth tech|
| API Scan mode:| cloud delivered SaaS, proactive scan through cloud APIs|
|               | data stored in the cloud|

### Forward Proxy
| **PROs** | **CONs** |
|---------|---------|
|should provide visibility and contrl over web traffic, allowing orgs to monitor and filter access to cloud apps and web apps| latency and additional complexity to network infrastructure|

---

# Virtual-Machine-Setup-and-Remote-Connection-Learning-Program
# Secure Remote Management & Cloud VM Hardening Lifecycle

## 1. Project Overview & Context

In modern cloud architecture and DevOps workflows, physical access to server hardware does not exist. Systems administrators, engineers, and developers must securely configure, maintain, and troubleshoot instances across public cloud boundaries using encrypted remote connectivity protocols. 

This repository documents the complete infrastructure lifecycle of an enterprise-tier cloud virtual machine running Microsoft Windows Server 2022 inside Microsoft Azure. The project begins with a baseline public-facing network topology and traces the identification of its structural vulnerabilities, credential rotation strategies, and subsequent architectural hardening. 

The primary deliverable demonstrates how an insecure management model (exposing raw Remote Desktop Protocol over public IP routing) is successfully transitioned into an enterprise-hardened posture using **Azure Bastion** to wrap operational sessions inside an encrypted, browser-encapsulated TLS/HTTPS tunnel.

### Learning Program Objectives Achieved
* **Virtual Network Topology Design**: Analyzed VNet address spacing and successfully applied network segmentation to create a dedicated gateway perimeter.
* **Firewall & Boundary Security**: Configured, modified, and audited stateful firewall configurations using Azure Network Security Groups (NSGs).
* **Dynamic Identity Remediation**: Applied platform extensions (`VMAccess`) to interact directly with the guest OS kernel and remediate administrative lockout issues.
* **Administrative Access Layer Isolation**: Shifted the platform management vector from insecure client-side tools (direct public RDP) to zero-trust web gateway connections.

---

## 2. Executive Summary

This engineering project highlights the critical security balance between infrastructure accessibility and perimeter hardening. The initial deployment profile consisted of a standard compute node provisioned with a dedicated Public IP address, exposing standard listening vectors (TCP Port 3389) across public pathways. During template synthesis, the Azure Resource Manager (ARM) engine threw explicit perimeter risk warnings highlighting the vulnerability of exposing raw RDP ports to the public internet.

Initial communication baseline testing encountered localized credential synchronization lag. Rather than using an destructive teardown/rebuild pattern, localized recovery was achieved by leveraging the Azure `VMAccess` extension to enforce a standardized administrator configuration (`azureuser`) within the guest OS. 

To systematically address the vulnerabilities associated with open management endpoints, the network configuration was refactored. A secure network slice, `AzureBastionSubnet` (`10.0.1.0/26`), was carved out of the root VNet topology to host a dedicated high-availability **Azure Bastion Host**. The final state successfully decouples administrative access from direct public routing. Operating sessions are handled cleanly over a TLS/HTTPS connection (`bastion.azure.com`), allowing secure operations through the native browser while isolating internal compute node interfaces from public scanning.

---

## 3. System Configuration Details & Metrics

The baseline compute environment, virtual network segmentation, and endpoint configurations were compiled and verified against the following infrastructure specifications:

### Core Infrastructure Metrics
* **Virtual Machine Identifier**: `Azure-WinServer`
* **Operating System Tier**: Windows Server 2022 Datacenter
* **Compute Profile Sizing**: `Standard D2ds v7` (2 vCPUs, 8.00 GiB RAM)
* **Parent Virtual Network (VNet)**: `vmhrwinastus201-vnet`
* **Resource Group Allocation**: `vmhrdevastus201_group`
* **Assigned Target Location**: `East US`

### Network Security Group (NSG) Inbound Rule Matrix
The network interface card (`azure-winserver373`) was associated with the `Azure-WinServer-nsg` stateful firewall. The core inbound rule matrix implemented across the project phases is detailed below:

| Priority | Rule Name | Port | Protocol | Source | Destination | Action | Phase / Description |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **300** | `Allow-Admin-Workstation` | `3389` | TCP | `102.219.128.28` | Any | **Allow** | Initial Baseline & Connectivity Testing |
| **65000** | `AllowVnetInBound` | Any | Any | `VirtualNetwork` | `VirtualNetwork` | **Allow** | Default Azure Internal East-West Core |
| **65001** | `AllowAzureLoadBalancerInBound` | Any | Any | `AzureLoadBalancer` | Any | **Allow** | Default Gateway Infrastructure Health Checking |
| **65500** | `DenyAllInBound` | Any | Any | Any | Any | **Deny** | Standard Fallback Edge Perimeter Dropping |

### Segmented Subnet Infrastructure
* **Workload Private Subnet (`default`)**: `10.0.0.0/24` $\rightarrow$ Dynamic Host Lease Address: `10.0.0.5`
* **Secure Perimeter Gateway Subnet (`AzureBastionSubnet`)**: `10.0.1.0/26` $\rightarrow$ Constraints: Exclusive use for Azure Bastion resources.

---

## 4. Step-by-Step Lab Implementation Report

### Phase 1: Pre-Deployment Architecture Verification
The lifecycle began during the resource template evaluation phase within the Azure Management Console. Upon initiating the "Review + create" process, the platform engine performed validation syntax checks. 
* **Status**: The workspace emitted a green **"Validation passed"** banner confirming resource structural integrity.
* **Security Assessment**: Concurrently, the ARM validation engine flagged an orange security alert: *"You have set RDP port(s) open to the internet. This is only recommended for testing..."* This verified the vulnerable baseline posture before configuration hardening began.

### Phase 2: Dependency Mapping & Deployment Progress
Upon confirming parameter inputs, execution was committed to `vmhrdevastus201_group`. Tracking the initial deployment logs provided visibility into the underlying platform resource dependency graph:
* The system systematically constructed network foundation components—the stateful Network Security Group (`Azure-WinServer-nsg`), the public address reservation (`4.157.46.197`), and the network adapter (`azure-winserver373`)—first.
* Once structural plumbing returned an **OK** confirmation status, the platform completed the compute engine creation phase, mounting the storage arrays and spinning up the `Azure-WinServer` virtual instance.

### Phase 3: Network Interface & Endpoint Discovery
Following compute instantiation, an audit of the asset essentials panel was performed to establish baseline operational metrics. 
* The primary virtual interface was verified as bound to the root virtual network network layout (`vmhrwinastus201-vnet`).
* The endpoints were registered and captured for the lab log: the public internet-routable IP address was recorded as `4.157.46.197`, and the corresponding internal host coordinate was set to `10.0.0.5`.

### Phase 4: Identity Remediation & Troubleshooting
Initial standard network access validation encountered localized credential synchronization errors (*"Your credentials did not work"*), causing a standard management lockout. 
* **Remediation Procedure**: To resolve this without forcing a destructive server rebuild, the administrative plane was accessed via the VM's connection recovery tools. The **"Reset password"** blade was triggered, leveraging the cloud runtime **VMAccess Extension** to interact with the active operating system kernel.
* The username was standardized to `azureuser`, and a robust password policy profile was pushed. The extension runtime successfully returned two automated **"ARM request succeeded"** execution receipts, indicating localized operating system profile synchronization was complete.
* Subsequent baseline connection checks via traditional client software threw standard untrusted security certificate alerts, documenting the high risk of sending administrative connections over unencrypted public paths.

### Phase 5: Network Segmentation & Bastion Provisioning
To systematically address the network vulnerabilities, the infrastructure architecture was updated to use a secure, air-gapped web gateway topology.
* **Subnet Segmentation**: The primary virtual network was modified to carve out an isolated address block explicitly meeting Azure's strict naming and parsing rules: **`AzureBastionSubnet`** mapped to network space `10.0.1.0/26`.
* **Host Provisioning**: The Bastion gateway configuration engine was configured with a **Standard Tier** architecture, utilizing an administrative scale count of `2` separate runtime instances to ensure zone-redundant high availability. The pre-flight engine confirmed validation success, and construction of the `vnet-bastion-host` was initiated.

### Phase 6: Browser-Encapsulated Verification & Guest OS Interaction
Once the edge gateway infrastructure shifted into a stable execution state, standard remote desktop application profiles were dropped entirely in favor of an identity-brokered perimeter session.
* An encrypted management tunnel query was triggered directly from the Azure portal interface, proxying traffic through an HTTPS browser handshake (`bastion.azure.com`).
* The guest operating system accepted the rotated administrative profile (`azureuser`). Successful interaction was verified by logging directly into the live Windows GUI container running inside the browser tab. The system search utility was verified as responsive, and the native **Windows Server Manager Dashboard** successfully initialized with full system-level visibility, proving deep, secure administration capabilities over an isolated network backplane.

### Phase 7: Post-Deployment Security Posture Audit
To finalize the deployment lifecycle, a structural review of the network configuration blades was performed.
* An audit of the VM's active **Connect** panel verified that the infrastructure setup is stable and fully supports secure enterprise connectivity profiles, highlighting active configurations for both **Bastion** and **Windows Admin Center** services.
* A standalone audit of the independent Network Security Group workspace confirmed that while the original test filtering rule remains restricted to the administrator's workstation, all subsequent operational session traffic runs entirely across protected internal virtual interfaces, ensuring a highly auditable and robust management perimeter.

---

## 5. Operational Recommendations

To ensure ongoing structural compliance with cloud security frameworks, the following operational adjustments are recommended for the production environment:

1.  **Decommission Vulnerable Rule Entries**: Delete the baseline high-priority rule (`Priority 300`) from `Azure-WinServer-nsg` to entirely shut down external port 3389 listening paths on the outer network border.
2.  **Enforce Just-In-Time (JIT) Network Policies**: If specific applications or deployment utilities require temporary direct access to public endpoints, couple those actions with Just-In-Time VM access configurations.
3.  **Implement Central Logging & Diagnostic Workspaces**: Configure streaming diagnostic logs for the active Azure Bastion host to point directly to an enterprise Azure Log Analytics Workspace. This ensures all active session creation events, connection duration metrics, and specific operator usernames are stored within an auditable, tamper-resistant log repository.



---
*Document compiled and verified for cloud security architecture portfolio review and academic submission.*

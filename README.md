# ☁️ Azure IAM & Blob Storage Lab — AZ-104 Hands-On Project

> **Author:** Collins Gora | Cloud Security & Azure Administration  
> **Certifications in Progress:** AZ-104 · AZ-500  
> **Lab Environment:** Microsoft Azure Portal (Live Subscription)  
> **Date Completed:** March 15, 2026  
> **Subscription:** Azure Subscription 1 | Tenant: `tichaonacollinsgoragmail.onmicrosoft.com`

---

## 📌 Project Overview

This project documents a **complete, end-to-end Azure Storage and Identity & Access Management (IAM) lab** performed on a live Azure subscription as part of my AZ-104 Microsoft Azure Administrator certification preparation. The lab covers the full lifecycle of secure storage administration — from provisioning a storage account in the Azure Portal, to configuring RBAC roles, assigning granular permissions at both the account and container level, uploading blobs, and reviewing the underlying JSON role definitions that power Azure's authorization model.

The work demonstrates real-world competence in:
- Provisioning and configuring Azure Storage Accounts
- Understanding and applying Azure RBAC (Role-Based Access Control)
- Differentiating between **Key-based** and **Entra ID (RBAC)** authentication for blob storage
- Assigning built-in roles (`Owner`, `Storage Blob Data Owner`, `Storage Blob Data Reader`) to users
- Working with IAM at both the **storage account scope** and **container scope**
- Uploading blobs and validating access through correct role assignments
- Inspecting JSON role definitions to understand the permission model behind Azure RBAC

---

## 🗂️ Table of Contents

1. [Architecture & Scope](#architecture--scope)
2. [Step 1 — Creating the Storage Account](#step-1--creating-the-storage-account)
3. [Step 2 — Exploring the Storage Account Overview](#step-2--exploring-the-storage-account-overview)
4. [Step 3 — Reviewing Storage Access Keys](#step-3--reviewing-storage-access-keys)
5. [Step 4 — Switching from Key Access to RBAC (Entra ID)](#step-4--switching-from-key-access-to-rbac-entra-id)
6. [Step 5 — Reviewing IAM Role Assignments at Storage Account Level](#step-5--reviewing-iam-role-assignments-at-storage-account-level)
7. [Step 6 — Exploring the Owner Role Permissions](#step-6--exploring-the-owner-role-permissions)
8. [Step 7 — Assigning Storage Blob Data Owner Role to Members](#step-7--assigning-storage-blob-data-owner-role-to-members)
9. [Step 8 — Verifying Role Assignments at Storage Account Scope](#step-8--verifying-role-assignments-at-storage-account-scope)
10. [Step 9 — Creating a Blob Container](#step-9--creating-a-blob-container)
11. [Step 10 — Assigning Roles at the Container Level (Granular IAM)](#step-10--assigning-roles-at-the-container-level-granular-iaM)
12. [Step 11 — Viewing Members Assigned at Container Scope](#step-11--viewing-members-assigned-at-container-scope)
13. [Step 12 — Uploading a Blob to the Container](#step-12--uploading-a-blob-to-the-container)
14. [Step 13 — Inspecting the JSON Role Definition](#step-13--inspecting-the-json-role-definition)
15. [Key Concepts & Takeaways](#key-concepts--takeaways)
16. [AZ-104 Exam Relevance](#az-104-exam-relevance)

---

## Architecture & Scope

```
Azure Subscription 1 (a34f347c-7958-47c8-9463-28b6f27b71a2)
│
└── Resource Group: cloud-shell-storage-westeurope
    │
    └── Storage Account: firstg
        ├── Location: South Africa North
        ├── Replication: RA-GRS (Read-Access Geo-Redundant Storage)
        ├── Performance: Standard
        ├── Account kind: StorageV2 (general purpose v2)
        │
        ├── IAM (Storage Account Scope)
        │   ├── Owner (Inherited from Subscription) — TICHAONA COLLINS GORA
        │   └── Storage Blob Data Owner — Collins Gora, Liyana Gora, TICHAONA
        │
        └── Containers
            ├── $logs  [system container — auto-created]
            └── myfirst  [created manually]
                ├── IAM (Container Scope)
                │   ├── Storage Blob Data Owner (inherited x3) — Collins, Liyana, TICHAONA
                │   └── Storage Blob Data Reader (this resource) — Lorraine Gora
                └── Blobs
                    └── IMG_1815.JPG  [4.43 MiB — Block Blob — Hot tier]
```

**Tenant Users Involved in Role Assignments:**

| Display Name | UPN | Role Assigned |
|---|---|---|
| TICHAONA COLLINS GORA | tichaonacollinsgoragmail@onmicrosoft.com | Owner (Subscription Inherited) |
| Collins Gora | CollinsGora@tichaonacollinsgoragmail.onmicrosoft.com | Storage Blob Data Owner |
| Liyana Gora | LiyanaGora@tichaonacollinsgoragmail.onmicrosoft.com | Storage Blob Data Owner |
| Loice Gora | loicegora@tichaonacollinsgoragmail.onmicrosoft.com | Storage Blob Data Owner |
| Lorraine Gora | lorrainegora@tichaonacollinsgoragmail.onmicrosoft.com | Storage Blob Data Reader |
| Gora (Guest) | goratichaona@gmail.com | N/A (guest account present in tenant) |

---

## Step 1 — Creating the Storage Account

> 📸 **Upload these screenshots into your `screenshots/` folder then they will display here automatically.**

![Storage Account Basics Form](screenshots/First_Azure_Resource_-_Storage.png)
*Filling in the storage account basics — name, region, performance, replication*

![Review and Create](screenshots/Create_Storage_Account.png)
*Review + Create tab confirming all settings before deployment*

The first task was to provision a new Azure Storage Account from the portal.

### Configuration Selected

| Setting | Value |
|---|---|
| Subscription | Azure Subscription 1 |
| Resource Group | cloud-shell-storage-westeurope |
| Storage Account Name | `firstg` |
| Region | South Africa North *(changed from default East US)* |
| Performance | Standard |
| Replication | Read-access geo-redundant storage (RA-GRS) |
| Access Tier | Hot |
| Hierarchical Namespace | Disabled |
| SFTP | Disabled |
| Infrastructure Encryption | Disabled |
| Minimum TLS | Version 1.2 |

### Why These Choices Matter

**Region — South Africa North:** Chosen deliberately to keep data sovereignty within South Africa. For AZ-104, understanding region selection and its impact on compliance, latency, and cost is critical.

**RA-GRS Replication:** This replication type keeps a primary copy in South Africa North and asynchronously replicates to a secondary region. The `Read-Access` prefix means the secondary endpoint is readable even without a failover being triggered — important for disaster recovery planning in the exam.

**Standard Performance:** Appropriate for general-purpose v2 accounts handling mixed workloads (blobs, files, queues, tables). Premium is only required for scenarios needing sub-millisecond latency.

**Hot Access Tier:** Optimized for data that is accessed frequently. Azure Blob Storage tiers (Hot, Cool, Archive) are a key AZ-104 topic — Hot has the highest storage cost but lowest access cost.

---

## Step 2 — Exploring the Storage Account Overview

![Storage Account Overview](screenshots/Storage_Resource.png)
*Storage account overview — essentials showing location, replication, provisioning state*

![Storage Blob Overview Properties](screenshots/Storage_Overview_Blob.png)
*Properties tab showing blob service settings and security configuration*

After deployment completed successfully, the storage account `firstg` was navigated to from the portal. The Overview blade confirms:

- **Provisioning State:** Succeeded
- **Account Kind:** StorageV2 (general purpose v2)
- **Blob Service Properties:**
  - Blob anonymous access: **Disabled** *(security best practice)*
  - Blob soft delete: **Enabled (7 days)** — protects against accidental deletion
  - Container soft delete: **Enabled (7 days)**
  - Versioning: **Disabled**
  - Change feed: **Disabled**
- **Security Settings:**
  - Require secure transfer (HTTPS): **Enabled**
  - Storage account key access: **Enabled**
  - Public network access: **Enabled**

> 🔐 **Security Note:** Blob anonymous access being disabled is a security baseline requirement. The AZ-500 exam tests this specifically — anonymous blob access can expose data to the public internet without any authentication.

---

## Step 3 — Reviewing Storage Access Keys

![Storage Access Keys Blade](screenshots/Storage_Access_Keys.png)
*Access Keys blade showing key1 and key2 with masked values and connection strings*

Navigated to **Security + Networking → Access Keys** on the storage account. Azure generates **two access keys** (`key1` and `key2`) for every storage account. These are symmetric shared secrets that grant **full control** over the storage account.

### Key Details Observed

- **Storage Account Name:** firstg
- **key1:** Hidden (masked) — Last rotated: 2026/03/14 (0 days ago)
- **key2:** Hidden (masked) — Last rotated: 2026/03/14 (0 days ago)
- **Connection strings** for both keys are available (for use in SDKs and Azure Storage Explorer)

### Why Two Keys?

The dual-key model exists to support **zero-downtime key rotation**. While rotating key1, applications can temporarily use key2, ensuring no service interruption. Azure recommends rotating keys regularly and storing them in **Azure Key Vault** rather than application config files.

### Key Access vs. RBAC — The Core Problem

Access keys bypass Azure RBAC entirely. Anyone with an access key has **full, unrestricted access** to all data in the storage account regardless of their Entra ID role assignments. This is why Microsoft recommends disabling key-based access where possible and using **Entra ID authentication (RBAC)** instead — it provides granular, auditable, identity-aware access control.

---

## Step 4 — Switching from Key Access to RBAC (Entra ID)

![IAM Role Assignments List](screenshots/IAM_Storage_role_assignemrnts.png)
*Access Control (IAM) showing the full list of available built-in storage roles including Storage Blob Data Contributor, Owner, and Reader*

Navigated to **Access Control (IAM)** on the storage account and reviewed the available built-in roles. The goal here was to move away from shared key authentication and begin assigning **RBAC roles** for data-plane access.

Azure provides three primary **data-plane** RBAC roles for Blob Storage:

| Role | Description | Use Case |
|---|---|---|
| **Storage Blob Data Contributor** | Read, write, and delete blobs and containers | Developers writing application data |
| **Storage Blob Data Owner** | Full access including POSIX ACL assignment | Admins needing full blob control |
| **Storage Blob Data Reader** | Read-only access to blob data | Auditors, read-only consumers |

> ⚠️ **AZ-104 Critical Distinction:** Even if a user has the `Owner` or `Contributor` role at the subscription or resource group level, they **cannot read blob data** unless they also have a `Storage Blob Data *` role OR key access. Management plane roles and data plane roles are separate in Azure Storage.

The portal also shows roles like `Storage Account Key Operator Service Role` (for managing keys), `Storage Account Contributor` (management-level control), and `Storage Blob Delegator` (for generating user delegation SAS tokens).

---

## Step 5 — Reviewing IAM Role Assignments at Storage Account Level

![Check Access — Owner Role Assignments](screenshots/Access_to_the_Storage_as_owner.png)
*Check Access panel showing TICHAONA COLLINS GORA has 2 Owner role assignments inherited from the subscription*

Opened **Access Control (IAM) → Check Access** and reviewed the role assignments for the account `TICHAONA COLLINS GORA`.

### Observed Role Assignments

| Role | Scope | Type | Notes |
|---|---|---|---|
| Owner | Subscription (Inherited) | — | Full management control inherited down |
| Owner | Subscription (Inherited) | — | Duplicate entry — both from same subscription |

**Key observation:** The `Owner` role is inherited from the subscription level. Azure RBAC uses a **hierarchical inheritance model**: permissions assigned at a higher scope (Management Group → Subscription → Resource Group → Resource) propagate down to child resources. This means owning the subscription means owning all resources within it, including this storage account.

**The error banner** (`An error occurred. Please try again later`) visible in the screenshot is a transient portal UI issue and does not indicate a configuration problem — role assignments were loading correctly.

---

## Step 6 — Exploring the Owner Role Permissions

![Owner Role Permissions Detail](screenshots/Owner_Priviledges_to_the_storage_account.png)
*Owner role definition blade showing 19,574 total permissions across all Microsoft resource providers — Actions tab selected*

Clicked on the **Owner** role to inspect its permission definitions. The Owner role details blade shows:

- **Description:** Grants full access to manage all resources, including the ability to assign roles in Azure RBAC.
- **Permission Type:** Actions (management plane)
- **Total Permissions:** 500 shown of **19,574 total** across all Microsoft resource providers
- **Scope:** Covers `Microsoft.AAD`, and all other resource providers

The permissions list includes critical actions such as:
- `Subscription Registration Action`
- `Register/Unregister Domain Service`
- `Read Domain Service`
- `Write/Delete Domain Service`

The **JSON tab** of any role definition reveals the raw `actions`, `notActions`, `dataActions`, and `notDataActions` arrays — this is how Azure evaluates effective permissions at request time.

> 📘 **AZ-104 Exam Note:** Owner is the most privileged built-in role. Unlike Contributor (which cannot assign roles), Owner can delegate access to others. Role assignment capability is what separates Owner from Contributor in Azure RBAC.

---

## Step 7 — Assigning Storage Blob Data Owner Role to Members

![Add Role Assignment — Select Members Panel](screenshots/Assign_member_to_role_blob_storage_data_owner.png)
*Members tab — selected role is Storage Blob Data Owner, panel showing all tenant users available for selection*

Navigated to **Access Control (IAM) → Add → Add role assignment** and went through the assignment workflow:

### Assignment Configuration

**Step 1 — Role Tab:**
- Selected **Storage Blob Data Owner**
- Description: *Allows for full access to Azure Storage blob containers and data, including assigning POSIX access control*

**Step 2 — Members Tab:**
- Assign access to: **User, group, or service principal**
- Clicked **+ Select members** which opened the member search panel

### Members Panel — Users Available in Tenant

The panel showed all users available in the `tichaonacollinsgoragmail.onmicrosoft.com` tenant:

| Account | Email |
|---|---|
| Accounts (service) | a370cdf9-3500-474e-b186-76c14feefe0e |
| Collins Gora | CollinsGora@tichaonacollinsgoragmail.onmicrosoft.com |
| Gora (Guest) | goratichaona@gmail.com |
| Liyana Gora | LiyanaGora@tichaonacollinsgoragmail.onmicrosoft.com |
| Loice Gora | loicegora@tichaonacollinsgoragmail.onmicrosoft.com |
| Lorraine Gora | lorrainegora@tichaonacollinsgoragmail.onmicrosoft.com |

Selected **Collins Gora** and **Liyana Gora** for the **Storage Blob Data Owner** role.

**Step 3 — Conditions Tab:** No conditions set (Conditions are an advanced ABAC feature for attribute-based access control — relevant for AZ-500).

**Step 4 — Review + Assign:** Confirmed the assignment.

> 💡 **Real-World Practice:** In enterprise environments, you would assign roles to **groups** rather than individual users to simplify management. Assigning directly to users is fine for lab purposes but creates overhead at scale.

---

## Step 8 — Verifying Role Assignments at Storage Account Scope

![Verified Role Assignments — 3 Total](screenshots/assigned_role_to_a_member.png)
*Check Access panel now showing 3 role assignments — 2 inherited Owner roles + newly assigned Storage Blob Data Owner at this resource scope*

After completing the role assignment, returned to **Access Control (IAM) → Check Access** and verified the updated assignments for the TICHAONA COLLINS GORA account.

### Updated Role Assignments (3 total)

| Role | Description | Scope | Condition |
|---|---|---|---|
| Owner | Grants full access to manage all resources | Subscription (Inherited) | None |
| Owner | Grants full access to manage all resources | Subscription (Inherited) | None |
| **Storage Blob Data Owner** | Allows for full access to Azure St… | **This resource** | Add |

The **Storage Blob Data Owner** role now shows scope as **"This resource"** — meaning it was assigned directly at the storage account level, not inherited. This is the correct scope for giving a user data-plane access to just this storage account without elevating their permissions on the entire subscription.

---

## Step 9 — Creating a Blob Container

![Containers List — myfirst created](screenshots/creating_a_container.png)
*Containers blade showing the newly created `myfirst` container alongside the system-created `$logs` container, both set to Private access*

Navigated to **Data Storage → Containers** and created a new container named `myfirst`.

### Containers Visible

| Container Name | Last Modified | Anonymous Access Level | Lease State |
|---|---|---|---|
| `$logs` | 2026/03/14 15:12:41 | Private | Available |
| `myfirst` | 2026/03/15 12:21:19 | Private | Available |

**`$logs`** is a system container automatically created by Azure to store Storage Analytics logs when logging is enabled. It is not user-created.

**`myfirst`** was manually created as the working container for this lab.

### Container Access Levels

When creating a container, you choose an **Anonymous access level**:

| Level | Description |
|---|---|
| **Private** | No anonymous access — Entra ID or key required |
| **Blob** | Anonymous read access for blobs only |
| **Container** | Anonymous read access for the container and all blobs within it |

`Private` was chosen here — the most secure option, requiring authenticated access for every request.

---

## Step 10 — Assigning Roles at the Container Level (Granular IAM)

![Add Role Assignment at Container Scope](screenshots/Assigning_role_in_a_container.png)
*Role list inside the myfirst container IAM blade — showing Storage Blob Data Contributor, Owner, and Reader as assignable at container scope*

Navigated to the `myfirst` container → **Access Control (IAM) → Add role assignment**. Azure supports RBAC at the **container scope** — a more granular level than the storage account. This allows you to grant a user access to one specific container without giving them any access to other containers in the same account.

### Roles Observed in the Container Role Assignment Panel

A subset of the roles visible at this scope:

| Role | Description | Type | Category |
|---|---|---|---|
| Storage Account Backup Contributor | Perform backup/restore via Azure Backup | BuiltInRole | Storage |
| Storage Account Contributor | Manage storage accounts (management plane) | BuiltInRole | Storage |
| Storage Account Key Operator Service Role | List and regenerate access keys | BuiltInRole | Storage |
| **Storage Blob Data Contributor** | Read, write, delete blobs and containers | BuiltInRole | Storage |
| **Storage Blob Data Owner** | Full access including POSIX ACL | BuiltInRole | Storage |
| **Storage Blob Data Reader** | Read-only blob access | BuiltInRole | Storage |
| Storage Blob Delegator | Generate user delegation SAS | BuiltInRole | Storage |

**Lorraine Gora** was assigned the **Storage Blob Data Reader** role at the container (`myfirst`) scope — meaning she can read blobs in this container but cannot write, delete, or access any other container in the storage account.

---

## Step 11 — Viewing Members Assigned at Container Scope

![Container IAM — All Members](screenshots/Assingned_members_.png)
*myfirst container Access Control (IAM) showing 11 total assignments — Owner inherited, Storage Blob Data Owner x3 inherited from parent, Storage Blob Data Reader assigned directly at this container*

After all assignments were complete, the **myfirst | Access Control (IAM)** view shows the full picture of who has access at the container level:

### Role Assignments — myfirst Container (11 results total)

**Owner (Subscription Inherited)**
| User | Type | Role | Scope |
|---|---|---|---|
| TICHAONA COLLINS GORA | User | Owner | Subscription (Inherited) |

**Storage Blob Data Owner (3 users)**
| User | Type | Role | Scope |
|---|---|---|---|
| Collins Gora | User | Storage Blob Data Owner | Parent resource (Inherited) |
| Liyana Gora | User | Storage Blob Data Owner | Parent resource (Inherited) |
| TICHAONA COLLINS | User | Storage Blob Data Owner | Parent resource (Inherited) |

**Storage Blob Data Reader (1 user)**
| User | Type | Role | Scope |
|---|---|---|---|
| Lorraine Gora | User | Storage Blob Data Reader | **This resource** |

**System Assignments (also visible):**
- Defender Agentless VM Scan (1)
- Defender CSPM Storage Scanner Operator (1)
- Defender Kubernetes API Access (1)
- Defender Sensitive Data Discovery (1)
- Defender Serverless Scanner (1)

> 🔍 **Key RBAC Insight:** Notice that `Collins Gora` and `Liyana Gora` show their Storage Blob Data Owner role as **"Parent resource (Inherited)"** — this is because the role was assigned at the **storage account level**, not the container level. RBAC inheritance flows downward, so the container automatically inherits it. `Lorraine Gora`'s Reader role shows **"This resource"** because it was assigned directly at the container scope.

---

## Step 12 — Uploading a Blob to the Container

![Upload Blob Panel — File Selected](screenshots/Upload_Blob_on_new_container_because_l_have_the_role.png)
*Upload blob panel inside myfirst container — IMG_1815.JPG selected, authentication method showing Microsoft Entra user account (not access key)*

![Blob Successfully Uploaded](screenshots/Uploaded_Blob_file.png)
*Container view after successful upload — IMG_1815.JPG visible as a Block Blob, 4.43 MiB, Hot tier, authenticated via Entra ID*

Navigated into the `myfirst` container and uploaded a blob. The upload was successful **because** the correct RBAC roles had been assigned.

### Upload Details

| Setting | Value |
|---|---|
| Container | myfirst |
| Authentication Method | **Microsoft Entra user account** *(not access key)* |
| File | IMG_1815.JPG |
| Blob Type | Block Blob |
| Size | 4.43 MiB |
| Access Tier | Hot (Inferred) |
| Upload Timestamp | 2026/03/15, 12:25:26 |

### Authentication Method — Entra vs. Access Key

The container overview shows: *Authentication method: Microsoft Entra user account (Switch to Access key)*

This is significant. The upload was authenticated using **Entra ID identity** (the logged-in user's Entra token), NOT a storage access key. This is the **correct, RBAC-aligned** method of accessing blob data — it creates an audit trail in Microsoft Entra ID logs and respects the RBAC assignments configured throughout this lab.

> ✅ **Proof of RBAC Working:** The fact that the upload succeeded confirms that the Storage Blob Data Owner role assigned to this user at the storage account level is functioning correctly. Without this role (or key access), the upload would have been denied with a 403 Forbidden error.

---

## Step 13 — Inspecting the JSON Role Definition

**Screenshot:** `Json_File_Azure_Container_Storage_Owner.png`

Navigated to the role definition JSON for **Azure Container Storage Owner** (a separate built-in role from Storage Blob Data Owner — this one manages ElasticSAN and container storage infrastructure).

### JSON Role Definition (Azure Container Storage Owner)

```json
{
  "id": "/providers/Microsoft.Authorization/roleDefinitions/95de85bd-744d-4664-9dde-11430bc34793",
  "properties": {
    "roleName": "Azure Container Storage Owner",
    "description": "Lets you install Azure Container Storage and grants access to its storage resources",
    "assignableScopes": [
      "/"
    ],
    "permissions": [
      {
        "actions": [
          "Microsoft.ElasticSan/elasticSans/*",
          "Microsoft.ElasticSan/locations/*",
          "Microsoft.ElasticSan/elasticSans/volumeGroups/*",
          "Microsoft.ElasticSan/elasticSans/volumeGroups/volumes/*",
          "Microsoft.ElasticSan/locations/asyncoperations/read",
          "Microsoft.KubernetesConfiguration/extensions/write",
          "Microsoft.KubernetesConfiguration/extensions/read",
          "Microsoft.KubernetesConfiguration/extensions/delete",
          "Microsoft.KubernetesConfiguration/extensions/operations/read",
          "Microsoft.Authorization/*/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Resources/subscriptions/read",
          "Microsoft.Management/managementGroups/read"
        ]
      }
    ]
  }
}
```

### Breaking Down the JSON Structure

| Field | Meaning |
|---|---|
| `id` | Unique resource ID of the role definition in Azure |
| `roleName` | Human-readable name of the role |
| `description` | What the role is designed to do |
| `assignableScopes: ["/"]` | Can be assigned at any scope (root = tenant/management group/subscription/RG/resource) |
| `actions` | Management plane operations allowed (control of Azure resources) |
| `dataActions` | Data plane operations (if any — not present here) |
| `notActions` | Excluded from the actions wildcard |
| `notDataActions` | Excluded from dataActions wildcard |

**Notable actions in this role:**
- `Microsoft.ElasticSan/elasticSans/*` — Full control of ElasticSAN resources
- `Microsoft.Authorization/*/read` — Can read all authorization (role definitions, assignments)
- `Microsoft.KubernetesConfiguration/extensions/*` — Can manage Kubernetes extensions (for AKS-based container storage)

> 📘 **AZ-104/AZ-500 Key Concept:** Every Azure RBAC role is ultimately defined as a JSON document. Understanding `actions` vs `dataActions` is critical — management plane actions control Azure resource metadata, while data actions control actual data. The Owner role uses `*` wildcards for both. Narrowly scoped roles like Storage Blob Data Reader use specific data action grants like `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`.

---

## Key Concepts & Takeaways

### 1. RBAC Scope Hierarchy
```
Management Group → Subscription → Resource Group → Resource → Sub-resource (Container)
```
Roles assigned at any level inherit downward. Assignments can also be made at the most granular level (container) to restrict blast radius.

### 2. Management Plane vs. Data Plane
- **Management plane** (`actions`): Controls Azure resource infrastructure — creating, deleting, configuring storage accounts.
- **Data plane** (`dataActions`): Controls actual data access — reading, writing, deleting blobs.
- A subscription-level `Owner` does NOT automatically get blob data access — you need an explicit `Storage Blob Data *` role.

### 3. Key Access vs. RBAC Authentication
| | Access Keys | Azure RBAC (Entra ID) |
|---|---|---|
| Audit trail | No identity tracking | Full Entra sign-in logs |
| Granularity | All or nothing | Fine-grained per role |
| Rotation | Manual (two keys) | Token-based, automatic |
| Recommended | Only for legacy/compat | ✅ Always preferred |

### 4. Inherited vs. Direct Role Assignments
- Roles assigned at storage account level show as **"Parent resource (Inherited)"** when viewed at container scope.
- Roles assigned directly at container scope show as **"This resource"**.
- Both are effective — but container-scoped assignments are more restrictive and precise.

### 5. Built-in Storage Roles Summary
| Role | Read Blob | Write Blob | Delete Blob | Assign Roles | Manage Account |
|---|---|---|---|---|---|
| Owner | ❌* | ❌* | ❌* | ✅ | ✅ |
| Storage Blob Data Reader | ✅ | ❌ | ❌ | ❌ | ❌ |
| Storage Blob Data Contributor | ✅ | ✅ | ✅ | ❌ | ❌ |
| Storage Blob Data Owner | ✅ | ✅ | ✅ | ✅ (POSIX) | ❌ |

> *Owner via management plane only — not blob data unless also assigned a data role or using access keys.

---

## AZ-104 Exam Relevance

This lab directly maps to the following AZ-104 exam objectives:

| Exam Domain | Topic Covered in This Lab |
|---|---|
| **Manage Azure identities and governance** | Azure RBAC, role assignments, scope hierarchy |
| **Implement and manage storage** | Storage account creation, replication options, access tiers |
| **Implement and manage storage** | Blob storage, containers, blob upload |
| **Implement and manage storage** | Storage access keys, SAS, RBAC auth methods |
| **Implement and manage storage** | IAM at storage account and container scope |
| **Monitor and maintain Azure resources** | Understanding Defender for Cloud assignments in IAM |

---

## 🛠️ Tools & Technologies Used

- Microsoft Azure Portal (portal.azure.com)
- Azure Blob Storage (StorageV2)
- Microsoft Entra ID (Azure Active Directory)
- Azure RBAC (Role-Based Access Control)
- Azure IAM (Identity and Access Management)
- Azure Storage Explorer (via portal browser)

---

## 📂 Repository Structure

```
├── README.md                          ← This file
├── screenshots/
│   ├── 01_First_Azure_Resource_Storage.png
│   ├── 02_Create_Storage_Account.png
│   ├── 03_Deployment_of_storage_complete.png
│   ├── 04_Storage_Resource.png
│   ├── 05_Storage_Overview_Blob.png
│   ├── 06_Storage_Access_Keys.png
│   ├── 07_IAM_Storage_role_assignments.png
│   ├── 08_Owner_Privileges_to_the_storage_account.png
│   ├── 09_Access_to_the_Storage_as_owner.png
│   ├── 10_Assign_member_to_role_blob_storage_data_owner.png
│   ├── 11_assigned_role_to_a_member.png
│   ├── 12_creating_a_container.png
│   ├── 13_Assigning_role_in_a_container.png
│   ├── 14_Assigned_members.png
│   ├── 15_Upload_Blob_on_new_container.png
│   ├── 16_Uploaded_Blob_file.png
│   └── 17_Json_File_Azure_Container_Storage_Owner.png
└── role-definitions/
    └── azure-container-storage-owner.json
```

---

## 🔗 References

- [Azure Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/)
- [Azure RBAC Built-in Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)
- [Authorize access to blobs using Entra ID](https://learn.microsoft.com/en-us/azure/storage/blobs/authorize-access-azure-active-directory)
- [AZ-104 Exam Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/)
- [Azure Storage redundancy options](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)

---

*This project is part of my AZ-104 Microsoft Azure Administrator certification lab series. Built on a live Azure subscription — no sandboxes, no simulations.*

**Collins Gora** | Cloud Security Professional | Cape Town, South Africa  
`AZ-900` ✅ | `MS-900` ✅ | `SC-900` ✅ | `Security+` ✅ | `AZ-104` 🔄 | `AZ-500` 🔄

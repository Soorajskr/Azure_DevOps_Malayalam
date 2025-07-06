# Session 07 – Terraform 02

# 📌 Backend Process – Resource Creation via Azure Portal vs Terraform

This document explains how Azure resources (like a Resource Group) are created using two methods:

- ✅ Azure Portal (GUI)
- ⚙️ Terraform (IaC)

---

## 🔷 Azure Portal Resource Creation Flow

```text
1️⃣  User opens Azure Portal
     → https://portal.azure.com

2️⃣  User logs in
     → Auth via Azure Entra (Azure AD)
     → Access token issued

3️⃣  User fills the form
     → Resource Group Name: my-rg
     → Location: East US

4️⃣  User clicks "Create"

5️⃣  Portal sends HTTPS PUT request to ARM
     → Includes access token
     → REST API: https://management.azure.com/...

6️⃣  Azure Resource Manager (ARM) receives request
     → Validates token
     → Checks RBAC (Permissions)
     → Validates request payload

7️⃣  ARM creates the Resource Group
     → Stores metadata in Azure

8️⃣  ARM sends success response
     → Status: 201 Created
     → Returns ID, Name, and Location

9️⃣  Portal refreshes view
     → my-rg now visible in Azure Portal

⚙️ Terraform Resource Creation Flow

1️⃣  User runs `terraform apply`
     → .tf file defines `azurerm_resource_group`

2️⃣  Terraform loads Azure provider
     → Reads provider block
     → Initializes plugin

3️⃣  Provider authenticates to Azure
     → Uses CLI / SPN / Env vars / Managed Identity
     → Gets access token from Azure AD

4️⃣  Provider converts HCL to REST API call
     → Prepares HTTPS PUT request
     → Adds token to headers
     → Payload includes RG name and location

5️⃣  ARM receives the request
     → Validates token and RBAC
     → Verifies payload structure

6️⃣  ARM provisions the Resource Group
     → my-rg is created

7️⃣  ARM sends response
     → HTTP 201 Created
     → Includes resource ID, name, and location

8️⃣  Terraform receives response
     → Confirms success
     → Marks resource as created

9️⃣  Terraform shows output
     → "Apply complete!" message
     → Lists created resources

🔟  Resource group appears in Azure Portal
     → my-rg shown in East US region

1️⃣1️⃣  Terraform updates `terraform.tfstate`
     → Saves resource ID, name, location
     → Used for future plan/apply/destroy
     → Stored locally or in remote backend
✅ Tip:
Terraform and the Azure Portal both use the same backend — the Azure Resource Manager (ARM) — through Azure REST APIs. The only difference is how the request is generated: manually (Portal) vs. automatically (Terraform).

## 🔍 Azure REST API

**URL:** `PUT https://management.azure.com/subscriptions/<sub-id>/resourcegroups/my-rg?api-version=2021-04-01`  
**REST API** = Representational State Transfer Application Programming Interface

> A REST API is like a waiter in a restaurant 🍽️  
> You (the client) ask the API (waiter) what you want.  
> The waiter talks to the kitchen (server) and brings back your food (data).

---

## 🔒 `.terraform.lock.hcl`

- Created with `terraform init`
- Prevents accidental upgrades
- Ensures version consistency, integrity & reproducibility

```hcl
provider "registry.terraform.io/hashicorp/azurerm" {
  version     = "4.34.0"
  constraints = "4.34.0"
  hashes = [
    "h1:Jf71pfqu9hEbKizPv1G6kds4MhB59T0TV+8wbwdi3TM=",
    "zh:8e43e2ad4f23cc8e0e1f51cdf19c0452ba97393958508e278a2bc135e28b2bbf"
  ]
}
Term	Meaning	Use
Hash	Digital fingerprint of a file	Proving file identity
Checksum	The actual hash value	Detecting tampering
In TF	Verifies downloaded plugin integrity	Ensures consistency

You can define version range like:
version = ">= 4.30.0, <= 4.35.0"

Upgrade command: terraform init -upgrade

Best Practices:

🎯 Pin exact version

🔐 Commit lock file

🧪 Stage upgrades

📚 Review changelogs

🔍 Review terraform plan

⚙️ CI/CD safety checks

🕵️ Track version history

🔄 Rollback strategy

🔐 Service Principal Authentication
Step 1: Create App Registration
Azure Portal → Entra ID → App registrations → + New registration → Name → Register

Step 2: Generate Client Secret
Certificates & Secrets → + New client secret → Add → Copy secret value

Step 3: Assign Role
Subscription → IAM → + Add role assignment → Contributor → Select App → Assign

CLI Method:
az ad sp create-for-rbac --name "TerraformSP" --role Contributor --scopes /subscriptions/<subscription-id>
Output:

json

{
  "appId": "xxxxx",
  "displayName": "TerraformSP",
  "password": "*****MASKED*****",
  "tenant": "xxxxx"
}
🔧 main.tf Provider Block

provider "azurerm" {
  features {}
  subscription_id = "xxxxx"
  tenant_id       = "xxxxx..."
  client_id       = "<appId>"
  client_secret   = "*****MASKED*****"
}
🖥️ Environment Variables (PowerShell)

$env:ARM_SUBSCRIPTION_ID="..."
$env:ARM_TENANT_ID="..."
$env:ARM_CLIENT_ID="..."
$env:ARM_CLIENT_SECRET="*****MASKED*****"
Permanent Set (Current User): powershell

[Environment]::SetEnvironmentVariable("ARM_TENANT_ID", "f885e5d7-d0af-43fd-a9a5-190468xxxxxx", "User")
[Environment]::SetEnvironmentVariable("ARM_CLIENT_ID", "a1325ce9-0f23-45d4-942e-654cfxxxxxxx", "User")
[Environment]::SetEnvironmentVariable("ARM_CLIENT_SECRET", "*****MASKED*****", "User")
📂 Using terraform.tfvars
Step 1: terraform.tfvars

subscription_id = "..."
tenant_id       = "..."
client_id       = "..."
client_secret   = "*****MASKED*****"
Step 2: Provider Reference

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
  client_id       = var.client_id
  client_secret   = var.client_secret
}
Step 3: Define Variables

variable "subscription_id" {}
variable "tenant_id" {}
variable "client_id" {}
variable "client_secret" {}
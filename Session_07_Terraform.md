# Session 07 – Terraform 02

## 📌 Backend Process – Resource Creation via Azure Portal vs Terraform

### 🔷 Azure Portal Flow

┌──────────────────────────────┐
│ 1. User Opens Azure Portal │
│ https://portal.azure.com │
└────────────┬────────────────┘
┌──────────────────────────────┐
│ 1. User Opens Azure Portal   │
│   https://portal.azure.com   │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 2. User Logs In │
│ - Auth via Azure Entra │
│ - Token issued │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 3. User fills form: │
│ - RG Name: my-rg │
│ - Location: East US │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 4. Clicks "Create" Button │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 5. Portal Sends HTTPS PUT │
│ REST API Request to ARM │
│ - Includes token │
│ - URL: https://management.│
│ azure.com/... │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 6. ARM Receives Request │
│ - Validates token │
│ - Checks RBAC │
│ - Validates payload │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 7. ARM Creates Resource Group│
│ - Stores metadata │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 8. ARM Sends Response │
│ - Status: Created │
│ - Resource ID, Name, Loc │
└────────────┬────────────────┘
▼
┌──────────────────────────────┐
│ 9. Portal Refreshes View │
│ - my-rg now visible │
└──────────────────────────────┘

---

### 🔷 Terraform Flow

┌─────────────────────────────────────────────────────┐
│ 1. User Runs terraform apply │
│ - .tf defines azurerm_resource_group │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 2. Terraform Loads azurerm Provider │
│ - Reads provider block │
│ - Initializes plugin │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 3. Provider Authenticates to Azure │
│ - CLI/SPN/Env/Managed Identity │
│ - Gets access token from Azure AD │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 4. Provider Translates HCL to REST API │
│ - Prepares HTTPS PUT request │
│ - Adds access token in headers │
│ - Payload includes RG name + location │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 5. Azure Resource Manager (ARM) Receives │
│ - Validates token with Azure AD │
│ - Checks RBAC & payload structure │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 6. ARM Provisions Resource Group │
│ - my-rg created │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 7. ARM Sends Response │
│ - HTTP 201 Created │
│ - Returns ID, name, location │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 8. Terraform Receives API Response │
│ - Verifies success │
│ - Marks resource as provisioned │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│ 9. Terraform Displays Output │
│ - Shows “Apply complete!” │
│ - Lists created resources │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│10. Resource Group Visible in Azure Portal │
│ - my-rg shown in East US │
└────────────────────────────┬────────────────────────┘
▼
┌─────────────────────────────────────────────────────┐
│11. Terraform Updates terraform.tfstate │
│ - Saves ID, name, location │
│ - Used in future plan/apply/destroy │
│ - Stored locally or remote backend │
└─────────────────────────────────────────────────────┘


---

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
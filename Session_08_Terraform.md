# Session 08 – Terraform Blocks

---

## 1. Required Providers Block

This block specifies the required providers, their sources, and versions:

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.35.0"
    }
  }
}
```

---

## 2. Provider Block

Configures how Terraform authenticates with Azure:

provider "azurerm" {
  features {}
  subscription_id = "44b09a79-b194-41d1-b9ef-b567ed97a565"
}
```

---

## 3. Resource Block

Creates resources:

resource "azurerm_resource_group" "example" {
  name     = "testrg_02"
  location = "West Europe"
}
```

### Required / Optional Arguments:

resource "azurerm_resource_group" "example" {
  # Required / Mandatory arguments
  name     = "testrg_02"
  location = "West Europe"

  # Optional argument – Tags (Map of arguments)
  tags = {
    Environment = "Test"
    Owner       = "Sooraj"
    Project     = "TerraformDemo"
  }
}
```

---

## Terraform Commands

### `terraform fmt` – Auto Formatting
- Standardizes indentation (2 spaces)
- Fixes alignment

### Why Proper Indentation Matters
- Improves **human readability**
- Enhances **maintainability**
- Helps **prevent errors**

---

## Terraform Validate

The primary command to check syntax and basic configuration:

terraform validate
```

### ✅ What It Checks:
- Syntax errors  
- Argument names and types  
- Required provider versions  
- Reference consistency  

---

## Apply & Destroy

terraform apply --auto-approve
terraform destroy --auto-approve
```

### 📌 What does `--auto-approve` do?

By default, Terraform prompts for confirmation before applying or destroying resources.  
Using `--auto-approve` skips this interactive approval step.

| Command                            | Behavior                                                                |
|-----------------------------------|-------------------------------------------------------------------------|
| `terraform apply`                 | Prompts: *"Do you want to perform these actions?"* → requires `yes`     |
| `terraform apply --auto-approve`  | Skips prompt and applies changes immediately                            |
| `terraform destroy`               | Prompts before destroying resources                                     |
| `terraform destroy --auto-approve`| Immediately destroys resources without confirmation                     |

### ⚠️ Use With Caution
- `--auto-approve` is useful for automation (CI/CD pipelines), but can be risky if not properly reviewed.
- Always verify the plan output before applying with `--auto-approve`.

---

## Commenting

- **Single line**: `#` or `//`
- **Multi-line**: `/* ... */`
- Shortcut: `Ctrl + /`

---

## 🔣 Meaning of Symbols in Terraform Plan

| Symbol | Meaning          | Description                                               |
|--------|------------------|-----------------------------------------------------------|
| `+`    | Create           | Terraform will create a new resource                      |
| `-`    | Destroy          | Terraform will delete the resource                        |
| `~`    | Update in-place  | Modify resource without replacing it                      |
| `-/+`  | Replace          | Delete and recreate the resource                          |
| `<=`   | Read/import      | Read from existing infrastructure (data source)           |

---

## Run Terraform Plan for Destroy

terraform plan -destroy
```

---

## 🔄 In-place Update (`~`)

📘 **Meaning**:  
Terraform modifies a property without recreating the resource.

✅ **Scenario**: Updating tags or fields that allow in-place changes.

🔧 **Example**:

resource "azurerm_resource_group" "example" {
  name     = "my-rg"
  location = "East US"
  tags     = {
    environment = "dev"
  }
}
```

**Change in Tags:**

```diff
~ tags = {
~   environment = "dev" -> "prod"
}
```

✅ Resource is updated in-place.

---

## 🔄 Replacement (`-/+`)

📘 **Meaning**:  
Terraform destroys and re-creates the resource (due to unsupported change).

❌ **Scenario**: Changing location of a resource group or IP of a NIC.

🔧 **Example**:

resource "azurerm_resource_group" "example" {
  name     = "my-rg"
  location = "East US"
}
```

**Change in Location:**

```diff
- location = "East US"
+ location = "West US"
```

❌ Azure doesn't support moving resource groups → recreation required.

---

## 🧠 Summary Table

| Change Type      | Symbol | Description                          | Example                          |
|------------------|--------|--------------------------------------|----------------------------------|
| In-place update  | `~`    | Changes supported without recreation | Updating tags                    |
| Replacement      | `-/+`  | Resource must be recreated           | Changing resource group location |

---

## Create Storage Account (First from Portal, then Terraform)

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}

resource "azurerm_storage_account" "example" {
  name                     = "storageaccountname"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "GRS"

  tags = {
    environment = "staging"
  }
}
```

---

## Create Two RGs and Two Storage Accounts

Tag each storage account in different resource groups.

---

## 📦 1. Account Tier (Performance Tier)

| Tier     | Description                                                                              |
|----------|------------------------------------------------------------------------------------------|
| Standard | HDD-based, cost-efficient for general use (blobs, queues, files, etc.)                   |
| Premium  | SSD-based, high-performance for production workloads (VM disks, DBs, etc.)               |

✅ Use **Standard** for logs/backups  
✅ Use **Premium** for low-latency production systems

---

## 🌍 2. Replication (Redundancy Options)

| Type   | Full Name                    | Description                                                        |
|--------|------------------------------|--------------------------------------------------------------------|
| LRS    | Locally Redundant Storage    | 3 copies in 1 datacenter – Cheapest                                |
| ZRS    | Zone-Redundant Storage       | 3 zones in a region – More resilient                               |
| GRS    | Geo-Redundant Storage        | Primary region + Secondary region                                  |
| GZRS   | Geo-Zone-Redundant Storage   | Combines ZRS + GRS                                                 |
| RA-GRS | Read-Access Geo-Redundant    | GRS + Read access to secondary                                     |

💡 Choose based on availability, cost, and disaster recovery needs.

---

## 🎯 Target a Specific Resource

resource "azurerm_resource_group" "example1" {
  name     = "testrg_03"
  location = "East US"

  tags = {
    Environment = "Prod"
  }
}
```

### Destroy Specific Resource:

terraform destroy -target="azurerm_resource_group.example1"
```

---

## 📁 Terraform Plan Output (`-out`)

Save plan to file and apply later:

terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
```

Destroy with saved plan:

terraform plan -destroy -out=tfplan
terraform apply tfplan
```

Confirm with:

terraform state list
```

---

## 🌟 Variables in Terraform

### ✅ What is a Variable?

A placeholder to dynamically assign values.

### 📌 Why Use Variables?

- Reusable code for multiple environments
- Customizable configuration
- Clean and flexible

---

## Types of Variables

| Term            | Description                            | Terraform Block     |
|------------------|----------------------------------------|---------------------|
| Input Variable   | Accepts user/system input              | `variable`          |
| Output Variable  | Exports values after `apply`           | `output`            |
| Local Value      | Internal reusable expression/value     | `locals`            |
| Module Input     | Input passed into a module             | `variable` in module |
| Module Output    | Output from a module                   | `output` in module   |

---

## 🧮 Variable Data Types

| Type    | Description                     | Example                     |
|---------|---------------------------------|-----------------------------|
| string  | Text                            | `"East US"`                |
| number  | Numeric                         | `3`, `3.14`                |
| bool    | Boolean                         | `true`                     |
| list    | Ordered list of same type       | `["a", "b"]`               |
| map     | Key-value pairs                 | `{ key = "value" }`        |
| set     | Unique unordered list           | `["a", "b"]`               |
| tuple   | Ordered list (mixed types)      | `["abc", 1, true]`         |
| object  | Structured values               | `{ name = "x", size = "S" }` |
| any     | Accepts any data type           | `"text"`, `[1, 2, 3]`      |

➡️ `list`, `map`, `set`, `tuple`, `object`, `any` — to be discussed later.

---

## 📦 Variable Block Example

variable "location" {
  type        = string
  default     = "East US"
  description = "The location where the resources will be created."
}

variable "resource_group" {
  type        = string
  default     = "test02"
  description = "The name of the resource group to create."
}

---

## 🎯 Using Variables in Resources

resource "azurerm_resource_group" "example" {
  name     = "testrg_01"
  location = var.location

  tags = {
    Environment = "Development"
  }
}
```

If no default is provided:

terraform apply -var="location=East US2"
```

---

✅ **End of Session 08**

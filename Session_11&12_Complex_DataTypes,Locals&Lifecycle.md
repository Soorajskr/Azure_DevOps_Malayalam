
# **Complex Data Types**
**List, Set, Tuple, Map, Object**

---
## **List**
- Contains elements of the same data type
- List can contain duplicate values
- **Example:**
    - `["abc", "def", "hij"]` – all strings
    - `[22, 34, 2, 5]` – all numbers
- A list is homogeneous

### **Array and index number (PowerShell Example):**
```powershell
$name = @("abc", "efg", "jkl")
$name[0]      # abc
$name[1]      # efg
$name.GetType()  # object[]
```

### **List Example in Terraform:**
```hcl
variable "users" {
    type = list(string)
    default = [ "user01","user02","user03" ]
}

resource "local_file" "file1" {
    filename = "./file1.txt"   #creating in current directory 
    content = var.users[0]
}

resource "local_file" "file2" {
    filename = "./file2.txt"
    content = var.users[1]
}

resource "local_file" "file3" {
    filename = "./file3.txt"
    content = var.users[2]
}
```

**Terraform console:**
```
terraform console
var.users
var.users[0]
```

- `list(any)` can also be used, but it should be same datatype  
- `list(object(...))` in Terraform: you can use all Terraform datatypes inside an object (string, number, bool, list, map, tuple, another object, etc.)

---
## **Tuple**
- Can contain different data types
- **Syntax:**
```hcl
type = tuple([string, number, bool])
```
- **Default value example:**
```hcl
["name", 20, true]
```
- It is heterogeneous (contains mixed data types)
- A tuple in Terraform always has an insertion order, and that order matters

**Example:**
```hcl
variable "items" {
    type = tuple([ string,number,bool ])
    default = [ "user01",56,true]
}

resource "local_file" "file1" {
    filename = "./file1.txt"
    content = var.items[0]
}

resource "local_file" "file2" {
    filename = "./file2.txt"
    content = var.items[1]
}

resource "local_file" "file3" {
    filename = "./file3.txt"
    content = var.items[2]
}
```

---
## **Set**
- Does not allow duplicate values and doesn’t follow the insertion order
- `for_each` supports sets
- If you want to use a list with `for_each`, convert it using the `toset()` function
- Unlike python, all datatype not supported in set
- **set(string)** → all elements must be strings
- **set(number)** → all elements must be numbers
- **set(object({...}))** → all elements must follow the same object structure (but inside the object you can have multiple types)

In Terraform, `set(any)` is a special type declaration that means:  
**A set of values that can be of any type, but all elements in that set must still be of the same type**

✅ **Example: Set of Strings**
```hcl
variable "fruit_set" {
  type    = set(any)
  default = ["apple", "banana", "cherry"]
}
variable "fruit_set" {
  type    = set(string)
  default = ["apple", "banana", "cherry"]
}
```

❌ **Example: Mixed types**
```hcl
variable "fruit_set" {
  type    = set(any)
  default = ["apple", 10, true]
}
```

---
## **Map**
A map is a collection of key-value pairs, similar to dictionaries in Python or objects in JavaScript.  
It’s used when you want to associate a value with a named key.

**Example:**
```hcl
variable "example_map" {
  type = map(string)
  default = {
    key1 = "value1"
    key2 = "value2"
  }
}
```

**Eg:**
```hcl
variable "tags" {
  type = map(string)  # or map(any)
  default = {
    environment = "prod"
    owner       = "sooraj"
  }
}

resource "azurerm_resource_group" "example" {
  name     = "my-rg"
  location = "East US"
  tags     = var.tags
}
```

**Eg of using only specific key:**
```hcl
resource "azurerm_resource_group" "example" {
  name     = "my-rg"
  location = "East US"

  tags = {
    environment = var.tags["environment"]
  }
}
```

**Terraform console:**
```
> var.tags
tomap({
  "environment" = "prod"
  "owner" = "sooraj"
})
> var.tags.environment
"prod"
```

---
## **Object**
An object in Terraform is a group of named values, where each name (key) has a fixed data type.  
It’s like a structured record.

**Eg:**
```hcl
variable "storage_config" {
  type = object({
    rg_name         = string
    location        = string
    storage_name    = string
    account_tier    = string
    account_replication_type = string
    tags            = map(string)
  })

  default = {
    rg_name         = "my-rg"
    location        = "East US"
    storage_name    = "mystorageacct1234"
    account_tier    = "Standard"
    account_replication_type = "LRS"
    tags = {
      env  = "dev"
      team = "infra"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = var.storage_config.rg_name
  location = var.storage_config.location
}

resource "azurerm_storage_account" "storage" {
  name                     = var.storage_config.storage_name
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = var.storage_config.account_tier
  account_replication_type = var.storage_config.account_replication_type
  tags                     = var.storage_config.tags
}
```

**Example of multiple datatype in object and nested object:**
```hcl
variable "vm_settings" {
  type = object({
    name               = string             # VM name
    location           = string             # Region
    vm_size            = string             # VM SKU
    instance_count     = number             # Number of VMs
    is_linux           = bool               # OS type
    tags               = map(string)        # Key-value tags
    dns_servers        = list(string)       # Custom DNS
    os_disk            = object({           # Nested object
      size_gb  = number
      type     = string
    })
  })

  default = {
    name           = "vm-app"
    location       = "East US"
    vm_size        = "Standard_B1s"
    instance_count = 2
    is_linux       = true
    tags = {
      env  = "dev"
      team = "backend"
    }
    dns_servers = ["8.8.8.8", "8.8.4.4"]
    os_disk = {
      size_gb = 64
      type    = "Standard_LRS"
    }
  }
}
```

---
## **Local Variable**
**What are locals in Terraform?**
- Locals are like temporary named values inside your Terraform configuration.
- They don’t take input from the user.
- They’re calculated within the configuration.
- They’re read-only — once defined, you can only use them, not change them.

**Example 1 - using locals**
```hcl
locals {
  rg_name  = "testrg"
  location = "East US"
}

resource "azurerm_resource_group" "example" {
  name     = local.rg_name
  location = local.location
}
```

**Example 2 - using locals**
```hcl
variable "env" {
  type    = string
  default = "prod"
}
locals {
  rg_name   = "${var.env}-resource-group" # in newer ver: rg_name = var.env + "-resource-group" (Interpolation)
  location  = "East US"
}

resource "azurerm_resource_group" "example" {
  name     = local.rg_name
  location = local.location
}
```

---
## **LIFECYCLE**
In Terraform, the `lifecycle` block is used to control how Terraform handles creation, updating, and deletion of a resource.  
It’s particularly useful in Azure when you have infrastructure that should not be accidentally destroyed or recreated, even if Terraform detects changes.  
It’s a **meta argument**.

### **Why use lifecycle?**
1. Preserve critical resources — e.g., Azure Storage Accounts holding production data.
2. Avoid downtime — prevent Terraform from replacing a resource when a small change is made.
3. Force recreation — tell Terraform to destroy and recreate a resource if a specific argument changes.
4. Ignore certain changes — if Azure automatically updates certain fields, you don’t want Terraform to mark them as “drift.”

### **Lifecycle Arguments**
| Argument               | Purpose                                          |
|------------------------|--------------------------------------------------|
| **prevent_destroy**    | Stops accidental deletion of the resource.       |
| **create_before_destroy** | Creates the new resource before deleting the old one (avoids downtime). |
| **ignore_changes**     | Ignores specified attributes when comparing config with the real resource. |

#### **Ignore change example**
If someone changes tags in the real Azure resource (outside Terraform), don’t try to fix it in the next `terraform apply`.  
The changes will remain as is. Terraform will:
- Still track the resource.
- Not treat those specific attribute changes as drift.
- Not plan any update for those attributes.

`ignore_changes` does **not** stop Terraform from updating the state file.  
It only tells Terraform **“Don’t plan changes to match config when this attribute differs.”**

### **Example Lifecycle Usage:**
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.35.0"
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = "44b09a79-b194-41d1-b9ef-b567ed97a565"
}

resource "azurerm_resource_group" "Label_RG" {
  name     = var.azurerm_resource_group
  location = var.location
}

resource "azurerm_storage_account" "Label_RG" {
  name                     = "datastore9099845"
  resource_group_name      = var.azurerm_resource_group
  location                 = var.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  depends_on               = [ azurerm_resource_group.Label_RG ]

  lifecycle {
    prevent_destroy       = true
    ignore_changes        = [ tags, access_tier ]
    create_before_destroy = true
  }
}
```

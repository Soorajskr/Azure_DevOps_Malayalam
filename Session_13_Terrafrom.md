# 🌟 Session 13 – Terraform Lifecycle & Data Sources

---

## 🔹 Lifecycle – Precondition

In Terraform, **`lifecycle → precondition`** is a **runtime validation check** that Terraform performs **before creating or updating** a resource.  

Think of it like a **safety gate** inside the resource definition —  
👉 If the condition isn’t met, Terraform will **stop** and show your **custom error message**.

### ✅ Example: Precondition
```hcl
resource "azurerm_resource_group" "myrg" {
  name     = var.resource_group_name
  location = var.location

  lifecycle {
    precondition {
      condition     = length(var.resource_group_name) <= 15
      error_message = "Resource Group name must be 15 characters or fewer."
    }

    precondition {
      condition     = endswith(var.resource_group_name, "-prod")
      error_message = "Resource Group name must end with '-prod'."
    }
  }
}

🔹 Lifecycle – Postcondition

Postcondition validates a resource’s state after creation or update.

Terraform creates/updates the resource as usual.

Afterward, Terraform checks your postcondition.

If condition ✅ passes → apply completes successfully.

If condition ❌ fails → Terraform will:

Mark the apply as failed.

Show the custom error message.

Leave the resource in Azure (❌ it doesn’t destroy automatically).

Mark resource as tainted → next terraform apply will recreate it.

✅ Example: Postcondition
resource "azurerm_resource_group" "example" {
  name     = "test-rg"
  location = "East US"

  lifecycle {
    postcondition {
      condition     = self.location == "eastus"
      error_message = "Resource group must be created in East US."
    }
  }
}

🔹 Data Sources in Terraform

A data source lets you read information about existing infrastructure from your provider (Azure, AWS, GCP, etc.) without creating or managing it.

📦 Data Block

Resource block → “Make me a new thing in Azure.”

Data block → “Show me details of a thing that already exists.”

data "azurerm_resource_group" "example" {
  name = "existing-RG"
}

output "rg_location" {
  value = data.azurerm_resource_group.example.location
}

🔹 Key Vault Example with Data Source

Imagine you already have a Key Vault in Azure.
Terraform won’t create it; it will just fetch its information.

🛠 Steps:

Create Key Vault & Secret in Azure Portal.

Set necessary IAM permissions:

Key Vault Administrator (full control; easiest while testing), OR

Key Vault Secrets Officer (create/update/delete secrets), OR

Key Vault Secrets User (read-only).

Assign access to:

Your user account (current login).

Service Principal used by Terraform.

🔹 Full Terraform Code Example

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

# --------------------------
# Resource Group
# --------------------------
resource "azurerm_resource_group" "rg" {
  name     = var.azurerm_resource_group
  location = var.location
}

# --------------------------
# Key Vault & Secret (Data Source)
# --------------------------
data "azurerm_key_vault" "example" {
  name                = "SoorajVaultAzure"
  resource_group_name = "kv-rg"
}

data "azurerm_key_vault_secret" "vm_password" {
  name         = "vm-admin-password"
  key_vault_id = data.azurerm_key_vault.example.id
}

# --------------------------
# Networking
# --------------------------
resource "azurerm_virtual_network" "vnet" {
  name                = "winvm-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = var.azurerm_resource_group
}

resource "azurerm_subnet" "subnet" {
  name                 = "winvm-subnet"
  resource_group_name  = var.azurerm_resource_group
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_security_group" "nsg" {
  name                = "winvm-nsg"
  location            = var.location
  resource_group_name = var.azurerm_resource_group

  security_rule {
    name                       = "RDP"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "3389"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_public_ip" "public_ip" {
  name                = "winvm-publicip"
  location            = var.location
  resource_group_name = var.azurerm_resource_group
  allocation_method   = "Dynamic"
  sku                 = "Basic"
}

# --------------------------
# Network Interface
# --------------------------
resource "azurerm_network_interface" "nic" {
  name                = "winvm-nic"
  location            = var.location
  resource_group_name = var.azurerm_resource_group

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.public_ip.id
  }
}

resource "azurerm_network_interface_security_group_association" "nsg_assoc" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}

# --------------------------
# Windows VM
# --------------------------
resource "azurerm_windows_virtual_machine" "winvm" {
  name                = "winvm01"
  location            = var.location
  resource_group_name = var.azurerm_resource_group
  network_interface_ids = [azurerm_network_interface.nic.id]
  size                = "Standard_DS1_v2"

  admin_username = var.admin_username
  admin_password = data.azurerm_key_vault_secret.vm_password.value # Must meet Azure complexity rules

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2019-Datacenter"
    version   = "latest"
  }

  computer_name            = "winvm01"
  provision_vm_agent       = true
  enable_automatic_updates = true
}

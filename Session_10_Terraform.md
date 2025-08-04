# 🚀 Windows Virtual Machine Creation on Azure with Terraform

This guide demonstrates how to deploy a **Windows Virtual Machine** in **Azure** using **Terraform**. It includes all necessary Azure resources such as:

- Resource Group  
- Virtual Network  
- Subnet  
- Network Security Group (NSG)  
- Public IP  
- Network Interface  
- Windows Virtual Machine  

---

## 📁 Variables


variable "azurerm_resource_group" {
  type    = string
  default = "winvm-rg"
}

variable "location" {
  type    = string
  default = "East US"
}
⚙️ Terraform Provider Configuration

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

  # Optional: You can use environment variables or Azure CLI authentication instead
  subscription_id = "44b09a79-b194-41d1-b9ef-b567ed97a565"
}
🏗️ Azure Resources
✅ Resource Group

resource "azurerm_resource_group" "rg" {
  name     = var.azurerm_resource_group
  location = var.location
}
🌐 Virtual Network

resource "azurerm_virtual_network" "vnet" {
  name                = "winvm-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = var.azurerm_resource_group
}
📶 Subnet

resource "azurerm_subnet" "subnet" {
  name                 = "winvm-subnet"
  resource_group_name  = var.azurerm_resource_group
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}
🔐 Network Security Group (NSG)

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
🌐 Public IP Address

resource "azurerm_public_ip" "public_ip" {
  name                = "winvm-publicip"
  location            = var.location
  resource_group_name = var.azurerm_resource_group
  allocation_method   = "Dynamic"
  sku                 = "Basic"
}
🧷 Network Interface

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
🔗 NSG Association with NIC

resource "azurerm_network_interface_security_group_association" "nsg_assoc" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id

  depends_on = [
    azurerm_windows_virtual_machine.winvm
  ]
}
💻 Windows Virtual Machine

resource "azurerm_windows_virtual_machine" "winvm" {
  name                  = "winvm01"
  location              = var.location
  resource_group_name   = var.azurerm_resource_group
  network_interface_ids = [azurerm_network_interface.nic.id]
  size                  = "Standard_DS1_v2"

  admin_username = "azureuser"
  admin_password = "************" # 👈 masked for GitHub-safe sharing

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

  computer_name             = "winvm01"
  provision_vm_agent        = true
  enable_automatic_updates  = true
}
📤 Terraform Outputs

output "vm_name" {
  description = "The name of the virtual machine"
  value       = azurerm_windows_virtual_machine.winvm.name
}

output "admin_username" {
  description = "The admin username for the virtual machine"
  value       = azurerm_windows_virtual_machine.winvm.admin_username
}

output "admin_password" {
  description = "The admin password for the virtual machine"
  value       = "************" # masked
  sensitive   = true
}

output "public_ip_address" {
  description = "The public IP address of the virtual machine"
  value       = azurerm_public_ip.public_ip.ip_address
}
📌 How to View Outputs
After applying the configuration, run:

terraform output
To view individual outputs:

terraform output vm_name
terraform output admin_username
terraform output admin_password
terraform output public_ip_address
🔗 depends_on Explanation
Terraform automatically handles most resource dependencies. However, when explicit control is needed (like ensuring the NIC NSG association happens after the VM is created), use:

depends_on = [ azurerm_windows_virtual_machine.winvm ]
This ensures that the NIC+NSG binding occurs only after the VM provisioning completes, avoiding potential race conditions.

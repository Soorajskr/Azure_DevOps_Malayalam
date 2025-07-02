
# Session 06: Terraform

## What is Terraform?

Terraform is an **open-source Infrastructure as Code (IaC)** tool developed by **HashiCorp**.

- A paid version is available called **Terraform Enterprise**, offering additional features.
- **No coding knowledge required.**
- Terraform code is **human-readable**, so there's no need to memorize code — just copy/paste from the [Terraform Registry](https://registry.terraform.io/).
- It supports a wide range of providers like **Azure, AWS, GCP, Alibaba**, and even **on-premises** platforms.

### Providers

A **provider** is a plugin that enables Terraform to interact with external platforms and services like:
- Azure
- AWS
- GCP
- GitHub
- VMware
- Kubernetes

> If you understand Terraform's logic, it's **the same across cloud providers**.

### Native IaC Tools in Cloud Providers

| Cloud Provider | Native IaC Tool               |
|----------------|-------------------------------|
| Azure          | ARM Templates, Bicep          |
| AWS            | CloudFormation                |
| GCP            | Google Cloud Deployment Manager |

---

## Language and Syntax

- Terraform uses **HashiCorp Configuration Language (HCL)** – a **declarative** language (key-value format).
- Terraform is developed using **Go (Golang)**.

### Declarative vs Imperative

| Style       | Description             | Example Tools          |
|-------------|-------------------------|------------------------|
| Declarative | "What you want"         | Terraform, Bicep       |
| Imperative  | "How to do it"          | Azure CLI, PowerShell  |

#### Real-Life Analogy

| Concept     | Real Life Example          | Tech Example              |
|-------------|----------------------------|---------------------------|
| Declarative | Ordering food from a menu  | Terraform                 |
| Imperative  | Cooking using a recipe     | Azure CLI / PowerShell    |

---

## Drawbacks

- ❌ No built-in **error handling mechanism**
- ❌ **Changes made via Azure Portal** are not auto-reflected in the Terraform state file
- ❌ **Corrupted `tfstate` file** can lead to infrastructure issues

> The `terraform.tfstate` file is a **critical component** — it maps real-world infra to your code.

---

## Notes & Setup Guide

All notes will be shared in the GitHub repository (link in the description).

---

## Install Terraform

- 📥 Link: [Terraform Install Guide](https://developer.hashicorp.com/terraform/install)

### Setup Steps

1. Extract the downloaded folder
2. Edit **System Environment Variables**
   - Add the extracted folder path to the **`PATH` variable**

#### Why Add to PATH?

So that the terminal can find and execute `terraform` commands globally.

---

## Environment Variable (EV)

An **environment variable** is a **name-value pair** storing system/application settings (e.g., paths, options, credentials).

---

## Install Prerequisites

### ✅ Azure CLI
Use to authenticate Terraform to Azure.

### ✅ Visual Studio Code
📥 [VS Code Download](https://code.visualstudio.com/download)

### Install Extensions:
- **HashiCorp Terraform**
- **Azure Terraform**

### Benefits of the VS Code Terraform Extension:
- ✅ Syntax highlighting  
- ✅ IntelliSense  
- ✅ Linting  
- ✅ Code formatting (`terraform fmt`)  
- ✅ Snippets (auto-templates)  
- ✅ Hover help & docs  
- ✅ Pre-apply validation  

---

## Search & Use Azure Resources from Registry

Terraform Registry is the **official online repo** for providers, modules, and docs.

### Resource Group Example

```hcl
resource "azurerm_resource_group" "example" {
  name     = "example"
  location = "West Europe"
}
```

---

## Provider and Initialization

### Authenticate to Azure via CLI

az login
```

### Terraform Configuration

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.34.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

> If you don't specify a version, Terraform installs the **latest** available:
```hcl
provider "azurerm" {
  features {}
}
```

---

# 🚀 Terraform Core Commands – Quick Reference

## 🛠️ `terraform init` — *Initialize Terraform Working Directory*

> Prepares your project for Terraform use by downloading required provider plugins and setting up the backend.

- Initializes `.terraform` directory
- Downloads the provider specified in `required_providers`
- Sets up remote backend (if configured)
- Creates `.terraform.lock.hcl` for version consistency

📌 Run this command **first** before `plan` or `apply`.

---

## 🧾 `terraform plan` — *Preview the Changes*

> Displays what Terraform **would do** (create, update, or delete resources) without applying any changes.

- Compares your `.tf` configuration with the current infrastructure and state
- Helps verify changes before applying
- Does **not** modify any resources

📌 Think of it as a **dry run** or **pre-check**.

---

## 🚀 `terraform apply` — *Apply the Changes*

> Executes the Terraform plan and **provisions** infrastructure described in your `.tf` files.

- Actually creates, modifies, or deletes resources
- Prompts for confirmation unless `-auto-approve` is used
- Updates the `terraform.tfstate` file after changes

📌 Think of it as **“make the changes live”**.

---

## 💣 `terraform destroy` — *Tear Down Everything*

> Destroys all resources **managed by Terraform** in the current working directory.

- Prompts for confirmation before deleting resources
- Removes all infrastructure defined in your Terraform project
- Useful for **decommissioning environments**

📌 Think of it as **“undo everything Terraform applied”**.

---

## 🔁 Common Terraform Workflow

```bash
terraform init       # Initialize providers & backend
terraform plan       # Preview what will change
terraform apply      # Apply those changes to the cloud
terraform destroy    # Delete everything (use with care!)





---





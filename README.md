# AZURE-VIRTUAL-MACHINE

This repository consists of Terraform templates to create Azure Virtual Machine object.

## Usage

- Clone this repo with: `git clone --recurse-submodules https://github.com/cklewar/azure-linux-virtual-machine`
- Enter repository directory with: `cd azure-linux-virtual-machine`
- Obtain F5XC API certificate file from Console and save it to `cert` directory
- Pick and choose from below examples and add mandatory input data and copy data into file `main.tf.example`.
- Rename file __main.tf.example__ to __main.tf__ with: `rename main.tf.example main.tf`
- Initialize with: `terraform init`
- Apply with: `terraform apply -auto-approve` or destroy with: `terraform destroy -auto-approve`

## Create Azure Linux Virtual Machine


```hcl
variable "project_prefix" {
  type        = string
  description = "prefix string put in front of string"
  default     = "f5xc"
}

variable "project_suffix" {
  type        = string
  description = "prefix string put at the end of string"
  default     = "01"
}

variable "azure_client_id" {
  type    = string
}

variable "azure_client_secret" {
  type    = string
}

variable "azure_tenant_id" {
  type    = string
}

variable "azure_subscription_id" {
  type    = string
}

variable "azure_region" {
  type    = string
  default = "eastus"
}

variable "ssh_public_key_file" {
  type    = string
}

provider "azurerm" {
  features {}
  client_id       = var.azure_client_id
  client_secret   = var.azure_client_secret
  tenant_id       = var.azure_tenant_id
  subscription_id = var.azure_subscription_id
  alias           = "eastus"
}

module "azure_resource_group" {
  source                    = "./modules/azure/resource_group"
  azure_region              = var.azure_region
  azure_resource_group_name = format("%s-azure-vm-rg-%s", var.project_prefix, var.project_suffix)
  providers                 = {
    azurerm = azurerm.eastus
  }
}

module "azure_virtual_network" {
  source                         = "./modules/azure/virtual_network"
  azure_region                   = var.azure_region
  azure_vnet_name                = format("%s-azure-vnet-%s", var.project_prefix, var.project_suffix)
  azure_vnet_primary_ipv4        = "172.16.24.0/21"
  azure_vnet_resource_group_name = module.azure_resource_group.resource_group["name"]
  providers                      = {
    azurerm = azurerm.eastus
  }
}

module "azure_subnet" {
  source                           = "./modules/azure/subnet"
  azure_subnet_address_prefixes    = ["172.16.24.0/24"]
  azure_subnet_name                = format("%s-azure-subnet-%s", var.project_prefix, var.project_suffix)
  azure_subnet_resource_group_name = module.azure_resource_group.resource_group["name"]
  azure_vnet_name                  = module.azure_virtual_network.vnet["name"]
  providers                        = {
    azurerm = azurerm.eastus
  }
}

module "azure_virtual_machine" {
  source                       = "./modules/azure/linux_virtual_machine"
  azure_region                 = var.azure_region
  azure_vnet_subnet_id         = module.azure_subnet.subnet["id"]
  azure_resource_group_name    = module.azure_resource_group.resource_group["name"]
  azure_virtual_machine_name   = format("%s-azure-vm-%s", var.project_prefix, var.project_suffix)
  azure_network_interface_name = format("%s-azure-vm-interface-%s", var.project_prefix, var.project_suffix)
  public_ssh_key               = file(var.ssh_public_key_file)
  providers                    = {
    azurerm = azurerm.eastus
  }
}

module "azure_security_group" {
  source                       = "./modules/azure/security_group"
  azure_region                 = var.azure_region
  azure_resource_group_name    = module.azure_resource_group.resource_group["name"]
  azure_security_group_name    = format("%s-spoke-a-sg-%s", var.project_prefix, var.project_suffix)
  azurerm_network_interface_id = element(module.azure_virtual_machine.virtual_machine ["network_interface_ids"], 0)
  azure_linux_security_rules   = [
    {
      name                       = "SSH"
      priority                   = 1001
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "22"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    },
    {
      name                       = "OUTBOUND_ALL"
      priority                   = 1002
      direction                  = "Outbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "*"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    }
  ]
  custom_tags = {
    "Owner" = var.owner_tag
  }
  providers = {
    azurerm = azurerm.eastus
  }
}

output "azure_virtual_machine" {
  value = module.azure_virtual_machine.virtual_machine
}
```
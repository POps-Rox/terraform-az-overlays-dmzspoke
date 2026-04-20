# Azure NoOps DMZ Spoke with all features deployed to Azure Government

This example is to create a Azure NoOps DMZ Spoke, with additional features.

```hcl
# Azure Provider configuration
provider "azurerm" {
  features {}
  skip_provider_registration = true
}

module "mod_vnet_spoke" {
  source  = "POps-Rox/overlays-dmz-spoke/azurerm"
  version = "~> 1.0"

  # By default, this module will create a resource group, provide the name here
  # To use an existing resource group, specify the existing resource group name, 
  # and set the argument to `create_resource_group = false`. Location will be same as existing RG.
  create_spoke_resource_group = true
  location                    = var.default_location
  deploy_environment          = var.deploy_environment
  org_name                    = var.org_name
  environment                 = var.environment
  workload_name               = var.dmz_name

  # Collect DMZ Virtual Network Parameters
  # DMZ network details to create peering and other setup
  hub_virtual_network_id          = data.azurerm_virtual_network.hub-vnet.id
  hub_firewall_private_ip_address = data.azurerm_firewall.hub-fw.ip_configuration[0].private_ip_address

  # To enable traffic analytics, set `enable_traffic_analytics = true` in the module.
  enable_traffic_analytics = var.enable_traffic_analytics

  # (Required) To enable Azure Monitoring and flow logs
  # pick the values for log analytics workspace which created by DMZ module
  # Possible values range between 30 and 730
  log_analytics_workspace_id           = data.azurerm_log_analytics_workspace.hub-logws.id
  log_analytics_customer_id            = data.azurerm_log_analytics_workspace.hub-logws.workspace_id
  log_analytics_logs_retention_in_days = 30

  # Provide valid VNet Address space for spoke virtual network.    
  virtual_network_address_space = var.dmz_vnet_address_space # (Required)  Spoke Virtual Network Parameters

  # (Required) Multiple Subnets, Service delegation, Service Endpoints, Network security groups
  # These are default subnets with required configuration, check README.md for more details
  # NSG association to be added automatically for all subnets listed here.
  # subnet name will be set as per Azure naming convention by default. expected value here is: <App or project name>
  spoke_subnets = var.dmz_subnets

  # Private DNS Zone Settings
  # If you do want to create additional Private DNS Zones, 
  # add in the list of private_dns_zones to be created.
  # else, remove the private_dns_zones argument.
  dmz_private_dns_zones = var.dmz_private_dns_zones

  # By default, this will apply resource locks to all resources created by this module.
  # To disable resource locks, set the argument to `enable_resource_locks = false`.
  enable_resource_locks = var.enable_resource_locks

  # Tags
  add_tags = local.tags # Tags to be applied to all resources
}


```

## Parameters Example Usage

```hcl
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

###########################
## Global Configuration  ##
###########################

# The prefixes to use for all resources in this deployment
org_name           = "anoa"         # This Prefix will be used on most deployed resources.  10 Characters max.
deploy_environment = "dev"          # dev | test | prod
environment        = "usgovernment" # public | usgovernment

# The default region to deploy to
default_location = "usgovvirginia"

# Enable locks on resources
enable_resource_locks = false # true | false

# Enable NSG Flow Logs
# By default, this will enable flow logs traffic analytics for all subnets.
enable_traffic_analytics = true

##########################################
#       DMZ Spoke Virtual Network      ###
##########################################

# DMZ Virtual Network Parameters
dmz_name               = "wl"
dmz_vnet_address_space = ["10.8.59.0/24"]
  untrusted = {
    name                                       = "untrusted"
    address_prefixes                           = ["10.8.59.0/27"]
    service_endpoints                          = ["Microsoft.Storage"]
    private_endpoint_network_policies_enabled  = false
    private_endpoint_service_endpoints_enabled = true
    
    nsg_subnet_rules = [
      {
        name                       = "Allow-Traffic-From-Spokes",
        description                = "Allow traffic from spokes",
        priority                   = 100,
        direction                  = "Inbound",
        access                     = "Allow",
        protocol                   = "Tcp",
        source_port_range          = "*",
        destination_port_range     = "443",
        source_address_prefix      = "*",
        destination_address_prefix = "*"
      }
    ]
  }
  trusted = {
    name                                       = "trusted"
    address_prefixes                           = ["10.8.59.64/27"]
    service_endpoints                          = ["Microsoft.Storage"]
    private_endpoint_network_policies_enabled  = false
    private_endpoint_service_endpoints_enabled = true
  }
  semitrusted = {
    name                                       = "semitrusted"
    address_prefixes                           = ["10.8.59.128/27"]
    service_endpoints                          = ["Microsoft.Storage"]
    private_endpoint_network_policies_enabled  = false
    private_endpoint_service_endpoints_enabled = true
    nsg_subnet_rules = []
  }
}

# Private DNS Zones
# Add in the list of private_dns_zones to be created.
dmz_private_dns_zones = []


```

## Terraform Usage

To run this example you need to execute following Terraform commands

```hcl
terraform init
terraform plan --var-file=parameters.tfvars --out dev.plan
terraform apply "dev.plan"
```

Run `terraform destroy` when you don't need these resources.

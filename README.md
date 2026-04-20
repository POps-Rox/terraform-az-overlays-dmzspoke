<!-- BADGES:START -->
[![CI](https://github.com/POps-Rox/tf-az-overlays-dmzspoke/actions/workflows/ci.yml/badge.svg)](https://github.com/POps-Rox/tf-az-overlays-dmzspoke/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/POps-Rox/tf-az-overlays-dmzspoke/pulls)
[![Maintained](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/POps-Rox/tf-az-overlays-dmzspoke/graphs/commit-activity)
[![Terraform](https://img.shields.io/badge/Terraform-%3E%3D1.5-623CE4.svg?logo=terraform)](https://www.terraform.io)
<!-- BADGES:END -->

# Azure DMZ Spoke Overlay Terraform Module

[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![MIT License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/POps-Rox/overlays-dmz-spoke/azurerm/)

This Overlay terraform module deploys a DMZ Spoke which is comprised of a virtual network and associated subnets following the [Microsoft recommended Hub-Spoke network topology](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) and conforming to the architecture of an [SCCA compliant Management Hub Network](https://registry.terraform.io/modules/POps-Rox/overlays-management-hub/azurerm/latest). This module can be used to create a Spoke in the same region as its Hub or it can be deployed to a different Azure region.  You can deploy it into the same subscription as the Hub or select a different one.

This module has a similar configuration to the Workload Spoke module except that it is designed to quickly deploy a DMZ spoke to support activities that need to happen just outside the Hub firewall, such as VPN termination. We recommend additional security hardening be applied to the network security group (NSG) deployed based on the security needs of the workload you are adding to this spoke.

## Using Different Azure Clouds

This module is built for use in all Azure Clouds (Public, US Government, Government Secret, and Government Top Secret). The [`environment`](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs#environment) and [`metada_host`](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs#metadata_host) variables are used by [Terraform's Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) to deploy to the appropriate Azure cloud. The following table shows the correct values for these variables for each non-air-gapped Azure cloud.  
| Azure Cloud | Environment | Metadata Host | Comments |
| ----------- | ----------- | ------------- | -------- |
| Commercial | "public" | not set | Default if `environment` not set |
| Government | "usgovernment" | not set | |
| Government Secret | value available in air-gap environment | value available in air-gap environment | |
| Government Top Secret | value available in air-gap environment | value available in air-gap environment | |
  
The `environment` variable is also closely associated with the `location` variable.  If you have `environment="public"` then you will need to set the `location` variable to one of the Commercial Azure Regions, for example `East US` or `eastus`. For `environment="usgovernment"` you would set the `location` variable to something like `usgovvirginia`

Example Usage for Azure Government Cloud:

```hcl

provider "azurerm" {
  environment = "usgovernment"
}

module "overlays-dmz-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "1.0.0"
  
  location = "usgovvirginia"
  environment = "usgovernment"
  ...
}

```

For more information on the Azure Government Cloud, see the [Azure Government](https://docs.microsoft.com/en-us/azure/azure-government/documentation-government-get-started) documentation.

## SCCA Compliance

This module deploys resources in an SCCA compliant manner and can be integrated into an existing SCCA compliant enclave. Enabling private endpoints and applying SCCA compliant network rules makes it SCCA compliant.

For more information, please read the [SCCA documentation]("https://www.cisa.gov/secure-cloud-computing-architecture").

## Contributing

If you want to contribute to this repository, please feel free to to contribute to our Terraform module.

More details are available in the [CONTRIBUTING.md](./CONTRIBUTING.md#pull-request-process) file.

## DMZ Spoke Architecture

The following reference architecture shows how to implement a SCCA compliant DMZ into a hub-spoke topology in Azure. The DMZ spoke virtual networks connect with the hub and can be used to isolate untrusted workloads. DMZ Spokes can exist in different subscriptions and support different types of untrusted workloads such as VPN or other encrypted transport tools.

The architecture below models an OpenVPN Encrypted Transport implementation withing a DMZ spoke.

![Architecture](<docs/img/Enclave_with_DMZ_Circled .png>)

These types of resources are supported:

* [Virtual Network](https://www.terraform.io/docs/providers/azurerm/r/virtual_network.html)
* [Subnets](https://www.terraform.io/docs/providers/azurerm/r/subnet.html)
* [Subnet Service Delegation](https://www.terraform.io/docs/providers/azurerm/r/subnet.html#delegation)
* [Virtual Network service endpoints](https://www.terraform.io/docs/providers/azurerm/r/subnet.html#service_endpoints)
* [Private Link service/Endpoint network policies on Subnet](https://www.terraform.io/docs/providers/azurerm/r/subnet.html#enforce_private_link_endpoint_network_policies)
* [AzureNetwork DDoS Protection Plan](https://www.terraform.io/docs/providers/azurerm/r/network_ddos_protection_plan.html)
* [Network Security Groups](https://www.terraform.io/docs/providers/azurerm/r/network_security_group.html)
* [Peering to Hub Network](https://www.terraform.io/docs/providers/azurerm/r/role_assignment.html)
* [Azure Monitoring Diagnostics](https://www.terraform.io/docs/providers/azurerm/r/monitor_diagnostic_setting.html)
* [Network Watcher](https://www.terraform.io/docs/providers/azurerm/r/network_watcher.html)
* [Network Watcher Workflow Logs](https://www.terraform.io/docs/providers/azurerm/r/network_watcher_flow_log.html)
* [Linking Hub Private DNS Zone](https://www.terraform.io/docs/providers/azurerm/r/private_dns_zone.html)

## Module Usage

```hcl
# Azurerm provider configuration
provider "azurerm" {
  features {}
}

data "azurerm_virtual_network" "hub-vnet" {
  name                = "anoa-eus-hub-core-dev-vnet"
  resource_group_name = "anoa-eus-hub-core-dev-rg"
}

data "azurerm_storage_account" "hub-st" {
  name                = "anoaeusd46f0d7ae4devst"
  resource_group_name = "anoa-eus-hub-core-dev-rg"
}

data "azurerm_log_analytics_workspace" "hub-logws" {
  name                = "anoa-eus-ops-mgt-logging-dev-log"
  resource_group_name = "anoa-eus-ops-mgt-logging-dev-rg"
}

module "vnet-wl-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "2.0.0"

  # By default, this module will create a resource group, provide the name here
  # To use an existing resource group, specify the existing resource group name, 
  # and set the argument to `create_resource_group = false`. Location will be same as existing RG.
  create_resource_group = true
  location              = "eastus"
  deploy_environment    = "dev"
  org_name              = "anoa"
  environment           = "public"
  workload_name         = "id-core"

  # Collect Hub Virtual Network Parameters
  # Hub network details to create peering and other setup
  hub_virtual_network_id          = data.azurerm_virtual_network.hub-vnet.id
  hub_firewall_private_ip_address = "10.0.100.4"  

  # (Required) To enable Azure Monitoring and flow logs
  # pick the values for log analytics workspace which created by Hub module
  # Possible values range between 30 and 730
  log_analytics_workspace_id           = data.azurerm_log_analytics_workspace.hub-logws.id
  log_analytics_customer_id            = data.azurerm_log_analytics_workspace.hub-logws.workspace_id
  log_analytics_logs_retention_in_days = 30

  # Provide valid VNet Address space for spoke virtual network.    
  virtual_network_address_space = ["10.0.100.0/24"] # (Required)  Hub Virtual Network Parameters
   
  # (Required) Multiple Subnets, Service delegation, Service Endpoints, Network security groups
  # These are default subnets with required configuration, check README.md for more details
  # Route_table and NSG association to be added automatically for all subnets listed here.
  # subnet name will be set as per Azure naming convention by default. expected value here is: <App or project name>
  spoke_subnets = {
    default = {
      name                                       = "id-core"
      address_prefixes                           = ["10.0.100.64/26"]
      service_endpoints                          = ["Microsoft.Storage"]
      private_endpoint_network_policies_enabled  = false
      private_endpoint_service_endpoints_enabled = true
    }
  }

  # Private DNS Zone Settings
  # By default, Azure NoOps will create Private DNS Zones for Logging in Hub VNet.
  # If you do want to create additional Private DNS Zones, 
  # add in the list of private_dns_zones to be created.
  # else, remove the private_dns_zones argument.
  private_dns_zones_to_link_to_hub = ["privatelink.file.core.windows.net"]  

  # By default, this will apply resource locks to all resources created by this module.
  # To disable resource locks, set the argument to `enable_resource_locks = false`.
  enable_resource_locks = false

  # Tags
  add_tags = {
    Example = "Workload Identity Core Spoke"
  } # Tags to be applied to all resources
}
```

## Spoke Networking

Spoke Networking is deployed against an existing SCCA Hub/Spoke architecture. The DMZ Spoke Overlay creates a Spoke virtual network with one or more subnets.  The virtual network is peered back to the existing Hub virtual network and NSG rules are created. Unlike the Workload and Management Spokes, no Forced Tunneling is implemented nor is a Route Table created by default. These will need to be added if you so desire.

The following parameters affect DMZ Spoke Overlay Networking.  

Parameter name | Location | Default Value | Description
-------------- | ------------- | ------------- | -----------
`virtual_network_address_space` | `variables.vnet.tf` | '10.0.100.0/24' | The CIDR Virtual Network Address Prefix for the Spoke Virtual Network.

## Subnets

This module handles the creation of subnets on the new virtual network.  The user passes a list of CIDR address spaces for the subnets. This module then uses a `for_each` to iterate over the list of CIDR addresses to create the requested subnets and corresponding service endpoints, service delegation, and network security groups. This module associates the subnets to network security groups which can also contain additional user-defined NSG rules.  

The module does not create a Default Subnet within the virtual network. The user must pass in the data for all subnets that are needed within the Spoke vnet.

## Virtual Network service endpoints

Service Endpoints allows connecting certain platform services into virtual networks.  With this option enabled, Azure virtual machines can interact with Azure PaaS services, such as Azure SQL or Azure Storage accounts, as-if they are sitting on the same private virtual network. Without the service endpoints being declared, any Azure virtual machines accessing the PaaS services will do so over the service's public endpoint (IP).  

You can configure this module to enable any or all of the following service endpoints on selected subnets in the virtual network. The list of Service endpoints available for association include: `Microsoft.AzureActiveDirectory`, `Microsoft.AzureCosmosDB`, `Microsoft.ContainerRegistry`, `Microsoft.EventHub`, `Microsoft.KeyVault`, `Microsoft.ServiceBus`, `Microsoft.Sql`, `Microsoft.Storage` and `Microsoft.Web`.

```hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
    default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"

      service_endpoints     = ["Microsoft.Storage"]  
    }
  }

# ....omitted

}
```

## Subnet Service Delegation

Subnet delegation lets you specify a subnet that will be used when you need to inject an Azure PaaS service into your virtual network. The Subnet delegation is the mechanism that allows you to fully manage the integration of Azure services into virtual networks.

This module supports enabling service delegation into a specific subnet under the spoke virtual network.  For more information, check the [terraform resource documentation](https://www.terraform.io/docs/providers/azurerm/r/subnet.html#service_delegation) or [What is subnet delegation?](https://learn.microsoft.com/en-us/azure/virtual-network/subnet-delegation-overview) on Microsoft Learn.

```hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
    default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"

      delegation = {
        name = "demodelegationcg"
        service_delegation = {
          name    = "Microsoft.ContainerInstance/containerGroups"
          actions = ["Microsoft.Network/virtualNetworks/subnets/join/action", "Microsoft.Network/virtualNetworks/subnets/prepareNetworkPolicies/action"]
        }
      }
    }
  }

# ....omitted

}
```

### `private_endpoint_network_policies_enabled` - Private Link Endpoint on the subnet

Network policies, like network security groups (NSG), are not supported for Private Link Endpoints. In order to deploy a Private Link Endpoint on a given subnet, you must set the `private_endpoint_network_policies_enabled` attribute to `true`. This setting is only applicable for the Private Link Endpoint, for all other resources in the subnet access is controlled via the Network Security Group, which can be configured using the `azurerm_subnet_network_security_group_association` resource.

This module can Enable or Disable network policies for the private link endpoints on the subnet. The default value is `false`. If you are enabling the Private Link Endpoints on the subnet then you shouldn't use Private Link Services as it will create conflicts.

```hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
   default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"
      private_endpoint_network_policies_enabled = true

        }
      }
    }
  }

# ....omitted
  
  } 
```

### `private_link_service_network_policies_enabled` - private link service on the subnet

In order to deploy a Private Link Service on a given subnet, you must set the `private_link_service_network_policies_enabled` attribute to `true`. This setting is only applicable for the Private Link Service. For all other resources in the subnet, access is controlled by the Network Security Group which can be configured using the `azurerm_subnet_network_security_group_association` resource.

This module can Enable or Disable network policies for the private link service on the subnet. The default value is `false`. If you are enabling the Private Link service on the subnet then you shouldn't use Private Link endpoints as it will create conflicts.

```hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
    default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"
      private_link_service_network_policies_enabled = true

        }
      }
    }
  }

# ....omitted

}
```
## Network Security Groups

Network security groups are defined to manage and log traffic on subnets. Use the `nsg_subnet_rules` block in this Terraform module to modify the Network Security Group (NSG) for each subnet with additional rules for inbound and outbound traffic.  The NSG will be created with Default rules and you can add additional rules to override the default behavior here.

The presence of the `nsg_subnet_rules` block is optional, but an NSG with default rules will be created and attached to the subnet regardless.  There is no way to create a subnet without an NSG from this module as NSGs are required for Network Flow Logs to operate and Flow Logs are required to meet SCCA requirements.

> For more information on the subnet NSG rule structure, see the [Azurerm NSG Terraform documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group#security_rule).


### Create a subnet with an NSG and default rules by omitting the `nsg_subnet_rules` block
``` hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
    default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"
    }
  }
}
```

### Create a subnet with an NSG that contains the default rules by adding an empty `nsg_subnet_rules` block

*You cannot remove the default rules, but you can override them by creating rules with higher priorities.*
``` hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
    default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"
      nsg_subnet_rules      = []
    }
  }
}
```

### Create a subnet with an NSG that contains the default rules and your custom rules
*You cannot remove the default rules, but you can override them by creating rules with higher priorities.*
``` hcl
module "vnet-spoke" {
  source  = "POps-Rox/tf-az-overlays-dmz-spoke/azurerm"
  version = "x.x.x"

  # .... omitted

  # Multiple Subnets, Service delegation, Service Endpoints
  subnets = {
    default = {
      subnet_name           = "default"
      subnet_address_prefix = "10.1.2.0/24"

      nsg_subnet_rules = [
      # Docs for the Security Rule block can be found at https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group#security_rule 
      {
        name                       = "allow-web-apps",
        description                = "Allow access to ports 80 & 443 for applications",
        priority                   = 100,
        direction                  = "Inbound",
        access                     = "Allow",
        protocol                   = "Tcp",
        source_port_range          = "*",
        destination_port_ranges    = ["80", "443"],
        source_address_prefix      = "*",
        destination_address_prefix = "*"
      },
      {
        name                       = "allow-ssh",
        description                = "Allow access to ports 22 for vm access",
        priority                   = 200,
        direction                  = "Inbound",
        access                     = "Allow",
        protocol                   = "*Tcp*",
        source_port_range          = "*",
        destination_port_range    = "22",
        source_address_prefix      = "*",
        destination_address_prefix = "*"      
      }
    ]
    }
  }
}
```






## Peering to the Hub virtual network

To peer spoke virtual networks to the hub virtual network requires the user/service principal that performs the peering to have the `Network Contributor` role on the hub virtual network. When linking the Spoke to the Hub DNS zones, the user/service principal also needs the `Private DNS Zone Contributor` role on the hub virtual network. If a Log Analytics workspace was created in the hub or another subscription then the user/service principal must also have the `Log Analytics Contributor` role on the workspace or a custom role to connect the new resources to the workspace.

> NOTE: This module will add the `Network Contributor` role and `Private DNS Zone Contributor` role, if you are using DNS Zones as part of the deployment.

## Create resource group

By default, this module will create a new resource group and name it based on the default naming convention. You can override the naming of the new resource group by passing your preferred name in the `custom_spoke_resource_group_name` argument. If you want to deploy into an existing resource group then you would pass the name of the existing resource group in the `existing_resource_group_name` argument and also set the `create_resource_group` argument to `false`.

> *If you are using an existing resource group then this module will create all resources in the same location (region) as the existing resource group provided.*

## Azure Network DDoS Protection Plan

This module uses the `create_ddos_plan` arrgument to enable or disable a DDOS Protection plan. By default, this module will not create a DDoS Protection Plan. If you want to enable a DDoS plan, set the `create_ddos_plan` argument to `true`. Azure only allows one DDoS Protection Plan per Subscription, so if you are deploying this Spoke into a Subscription that already has a DDoS Plan, please leave the default of false to avoid a runtime error.

## Azure Network Network Watcher

This module let's Azure handle the provisioning of Network Watcher resources. By default, this module will enable flow logs and traffic analytics for all the subnets in the Virtual Network.Any resources created will be placed into the `NetworkWatcherRG` resource group.

> **Note:** *A Log Analytics workspace is required for NSG Flow Logs and Traffic Analytics. If you want to enable NSG Flow Logs and Traffic Analytics, you must create a Log Analytics workspace and provide the workspace name in the `log_analytics_workspace_name` argument and the workspace's resource group name in the `log_analytics_workspace_resource_group_name` argument.*

## Custom DNS servers

This is an optional feature and only applicable if you are using your own DNS servers superseding default DNS services provided by Azure. Set the argument `dns_servers = ["4.4.4.4"]` to enable this option. For multiple DNS servers, set the argument `dns_servers = ["4.4.4.4", "8.8.8.8"]`

## Linking Hub Private DNS Zone

This module facilitates linking the spoke VNet to private DNS, preferably created by the Spoke Module. To create a link to a private DNS zone, providet the domain name of the private DNS zone to the `private_dns_zones` argument. If you want to link multiple private DNS zones, provide the list of DNS Zones to the `private_dns_zones` argument like this: `private_dns_zones = ["privatelink.blob.core.windows.net", "privatelink.file.core.windows.net"]`  

## Recommended naming and tagging conventions

You can apply tags to your Azure resources, resource groups, and subscriptions to logically organize them into a taxonomy. Each tag consists of a name and a value pair. For example, you can apply the name `Environment` and the value `Production` to all the resources in production.
For recommendations on how to implement a tagging strategy, see the [Resource naming and tagging decision guide](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming-and-tagging-decision-guide).

>**Important** :
Tag names are _case-insensitive_ whereas tag values are _case-sensitive_. The resource provider might keep the casing you provide for the tag name. You'll see that casing in cost reports.

An effective naming convention assembles resource names by using important resource information as parts of a resource's name. For example, using these [recommended naming conventions](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#example-names), a public IP resource for a production SharePoint workload is named like this: `pip-sharepoint-prod-westus-001`.

## Other resources

* [Hub-spoke network topology in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
* [Terraform AzureRM Provider Documentation](https://www.terraform.io/docs/providers/azurerm/index.html)

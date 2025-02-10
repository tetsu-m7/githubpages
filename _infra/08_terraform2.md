---
permalink: /infra/terraform2
layout: single
title: "Terraform 2"
---

# はじめに

Terraform をConnection Monitor つきで多少Module 化したものを備忘録として記載します。

# フォルダ構造

{% highlight bash %}
.
├── envs
│   └── dev
│       ├── main.tf
│       ├── outputs.tf
│       ├── terraform.tfstate
│       ├── terraform.tfvars
│       └── variables.tf
├── modules
│   ├── compute
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   └── monitoring
│       ├── main.tf
│       ├── outputs.tf
│       └── variables.tf
{% endhighlight %}

# main.tf 

{% highlight bash %}
provider "azurerm" {
  features {}
}

module "dev-alma1" {
  source          = "../../modules/compute"
  vm_name         = "dev-alma1"
  resource_group  = var.resource_group
  location        = var.location
  vnet_name       = var.vnet_name
  subnet_name     = var.subnet_name
  admin_username  = var.admin_username
  ssh_public_key  = file(var.ssh_public_key_path)
  alma_version    = var.alma_version
  vm_size         = var.vm_size
}
module "dev-alma2" {
  source          = "../../modules/compute"
  vm_name         = "dev-alma2"
  resource_group  = var.resource_group
  location        = var.location
  vnet_name       = var.vnet_name
  subnet_name     = var.subnet_name
  admin_username  = var.admin_username
  ssh_public_key  = file(var.ssh_public_key_path)
  alma_version    = var.alma_version
  vm_size         = var.vm_size
}

resource "azurerm_virtual_machine_extension" "network_watcher" {
  name                 = "NetworkWatcherAgent"
  virtual_machine_id   = module.dev-alma1.vm_id
  publisher            = "Microsoft.Azure.NetworkWatcher"
  type                 = "NetworkWatcherAgentLinux"
  type_handler_version = "1.4"
  auto_upgrade_minor_version = true

  depends_on = [module.dev-alma1] # VM 作成後に実行
}

resource "azurerm_resource_group" "res-0" {
  location = "westus"
  name     = "NetworkWatcherRG"
}
resource "azurerm_network_watcher" "res-1" {
  location            = "eastus"
  name                = "NetworkWatcher_eastus"
  resource_group_name = "NetworkWatcherRG"
  depends_on = [
    azurerm_resource_group.res-0,
  ]
}
resource "azurerm_network_watcher" "res-2" {
  location            = "westus"
  name                = "NetworkWatcher_westus"
  resource_group_name = "NetworkWatcherRG"
  depends_on = [
    azurerm_resource_group.res-0,
  ]
}
resource "azurerm_network_connection_monitor" "res-3" {
  location           = "eastus"
  name               = "connection_Monitor_eastus"
  network_watcher_id = "/subscriptions/xxxxx-xxxxx-xxxx-xxxxxx/resourceGroups/networkwatcherrg/providers/Microsoft.Network/networkWatchers/NetworkWatcher_eastus"
  endpoint {
    name                 = "dev-alma1(terraform)"
    target_resource_id   = "/subscriptions/xxxxx-xxxxx-xxxx-xxxxxx/resourceGroups/terraform/providers/Microsoft.Compute/virtualMachines/dev-alma1"
  }
  endpoint {
    name                 = "dev-alma2(terraform)"
    target_resource_id   = "/subscriptions/xxxxx-xxxxx-xxxx-xxxxxx/resourceGroups/terraform/providers/Microsoft.Compute/virtualMachines/dev-alma2"
  }
  test_configuration {
    name                      = "pingTest"
    protocol                  = "Icmp"
    test_frequency_in_seconds = 30
    icmp_configuration {
    }
    success_threshold {
      checks_failed_percent = 90
      round_trip_time_ms    = 100
    }
  }
  test_configuration {
    name                      = "port80monitoring"
    protocol                  = "Tcp"
    test_frequency_in_seconds = 30
    success_threshold {
      checks_failed_percent = 90
      round_trip_time_ms    = 100
    }
    tcp_configuration {
      port = 80
    }
  }
  test_group {
    destination_endpoints    = ["dev-alma2(terraform)"]
    name                     = "dev-alma2Monitoring"
    source_endpoints         = ["dev-alma1(terraform)"]
    test_configuration_names = ["pingTest", "port80monitoring"]
  }
  depends_on = [
    azurerm_network_watcher.res-1,module.dev-alma1,module.dev-alma2
  ]
}
resource "azurerm_monitor_metric_alert" "res-4" {
  name                = "connection_Monitor_eastus"
  resource_group_name = "networkwatcherrg"
  scopes              = ["/subscriptions/xxxxx-xxxxx-xxxx-xxxxxx/resourceGroups/networkwatcherrg/providers/Microsoft.Network/networkWatchers/NetworkWatcher_eastus/connectionMonitors/connection_Monitor_eastus"]
  action {
    action_group_id = "/subscriptions/xxxxx-xxxxx-xxxx-xxxxxx/resourcegroups/test/providers/microsoft.insights/actiongroups/alert-email"
  }
  criteria {
    aggregation      = "Maximum"
    metric_name      = "TestResult"
    metric_namespace = "Microsoft.Network/networkWatchers/connectionMonitors"
    operator         = "GreaterThan"
    threshold        = 2
    dimension {
      name     = "SourceName"
      operator = "Include"
      values   = ["*"]
    }
    dimension {
      name     = "DestinationName"
      operator = "Include"
      values   = ["*"]
    }
    dimension {
      name     = "TestGroupName"
      operator = "Include"
      values   = ["*"]
    }
    dimension {
      name     = "TestConfigurationName"
      operator = "Include"
      values   = ["*"]
    }
  }
  depends_on = [
    azurerm_network_connection_monitor.res-3,
  ]
}

{% endhighlight %}

# Module compute

{% highlight bash %}
resource "azurerm_public_ip" "vm_pip" {
  name                = "${var.vm_name}-pip"
  location            = var.location
  resource_group_name = var.resource_group
  allocation_method   = "Static"
  sku                 = "Basic"
}

resource "azurerm_network_interface" "vm_nic" {
  name                = "${var.vm_name}-nic"
  location            = var.location
  resource_group_name = var.resource_group

  ip_configuration {
    name                          = "internal"
    subnet_id                     = data.azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.vm_pip.id
  }
}

data "azurerm_subnet" "subnet" {
  name                 = var.subnet_name
  virtual_network_name = var.vnet_name
  resource_group_name  = var.resource_group
}

resource "azurerm_virtual_machine" "vm" {
  name                  = var.vm_name
  location              = var.location
  resource_group_name   = var.resource_group
  network_interface_ids = [azurerm_network_interface.vm_nic.id]
  vm_size               = var.vm_size
  delete_os_disk_on_termination = true

  storage_os_disk {
    name              = "${var.vm_name}-osdisk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  os_profile {
    computer_name  = var.vm_name
    admin_username = var.admin_username
  }

  os_profile_linux_config {
    disable_password_authentication = true
    ssh_keys {
      path     = "/home/${var.admin_username}/.ssh/authorized_keys"
      key_data = var.ssh_public_key
    }
  }

  storage_image_reference {
    publisher = "almalinux"
    offer     = "almalinux-x86_64"
    sku       = "9-gen2"
    version   = var.alma_version
  }
}


{% endhighlight %}

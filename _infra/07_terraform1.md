---
permalink: /infra/terraform1
layout: single
title: "Terraform 1"
---

# はじめに

このドキュメントはTerraform 初心者がAzure上にリソースをDeployする方法を記載します。  
環境としてはWindows を利用している前提で、WSL 上にTerraform をインストールし、ファイルを準備し、Deployまで実施します。  
基本的にはこのドキュメントを上から下に流し込めば、Azure 上にStorage accountの作成とLinux OS のDeploy ができることを目的とします。
なお、Azure の環境は多くの場合特定のResourceを指定することが多いと思うので、Subscription / vNet / Subnet については既存のものを指定する形で作成しています。適宜必要があれば書き換えて利用してください。

# Terraform をWSLにインストール

{% highlight bash %}
sudo apt-get update
sudo apt install -y git curl unzip

git clone https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

tfenv install latest
tfenv use latest

terraform -v
{% endhighlight %}

# Terraform を利用するためのAzure側の設定

{% highlight bash %}
az ad sp create-for-rbac --name "terraform-service-principal" --role="Contributor" --scopes="/subscriptions/Pay-As-You-Go"
{% endhighlight %}

# 課金を起こさない無害なStorage accountを作る方法

* param.tfvars
{% highlight bash %}
provider_credentials = {
  subscription_id  = "<subscription id>>"
  tenant_id        = "＜Microsoft Entra ID＞"
  sp_client_id     = "＜クライアントID＞"
  sp_client_secret = "＜クライアントシークレット＞"
}
{% endhighlight %}
* main.tf
{% highlight bash %}
provider "azurerm" {
  subscription_id = var.provider_credentials.subscription_id
  tenant_id       = var.provider_credentials.tenant_id
  client_id       = var.provider_credentials.sp_client_id
  client_secret   = var.provider_credentials.sp_client_secret
  features {}
}

locals {
  resource_group_name       = "test-resourcegroup"
  storage_account_diag_name = "testblob20241215"
}

resource "azurerm_resource_group" "je" {
  name     = local.resource_group_name
  location = var.location
}

# 診断ログの保管ストレージ
resource "azurerm_storage_account" "diag" {
  name                     = local.storage_account_diag_name
  resource_group_name      = azurerm_resource_group.je.name
  location                 = azurerm_resource_group.je.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
{% endhighlight %}
* variables.tf
{% highlight bash %}
variable "provider_credentials" {
  type = object({
    subscription_id  = string
    tenant_id        = string
    sp_client_id     = string
    sp_client_secret = string
  })
}

variable "location" {
  type    = string
  default = "japaneast"
}
{% endhighlight %}
* 作成
{% highlight bash %}
terraform init
terraform plan -var-file=param.tfvars
terraform apply -var-file=param.tfvars
{% endhighlight %}

* 削除
{% highlight bash %}
terraform destroy -var-file=param.tfvars
{% endhighlight %}

# Alma Linux 8系の最新版を立ててみよう

* main.tf
{% highlight bash %}
provider "azurerm" {
  subscription_id = var.provider_credentials.subscription_id
  tenant_id       = var.provider_credentials.tenant_id
  client_id       = var.provider_credentials.sp_client_id
  client_secret   = var.provider_credentials.sp_client_secret
  features {}
}
# 既存のリソースグループを参照
data "azurerm_resource_group" "existing_rg" {
  name = "test"   # 既存のリソースグループ名
}

# 既存のVNetとSubnetを参照
data "azurerm_virtual_network" "existing_vnet" {
  name                = "vNet1"  # 既存のVNet名
  resource_group_name = data.azurerm_resource_group.existing_rg.name
}

data "azurerm_subnet" "existing_subnet" {
  name                 = "default" # 既存のSubnet名
  virtual_network_name = data.azurerm_virtual_network.existing_vnet.name
  resource_group_name  = data.azurerm_virtual_network.existing_vnet.resource_group_name
}

# パブリックIPの作成
resource "azurerm_public_ip" "test_public_ip" {
  name                = "test-public-ip"
  location            = data.azurerm_resource_group.existing_rg.location
  resource_group_name = data.azurerm_resource_group.existing_rg.name
  allocation_method   = "Static"
}

# 仮想ネットワークインターフェースカード（NIC）
resource "azurerm_network_interface" "test_nic" {
  name                = "test-nic"
  location            = data.azurerm_resource_group.existing_rg.location
  resource_group_name = data.azurerm_resource_group.existing_rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = data.azurerm_subnet.existing_subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.test_public_ip.id
  }
}
# セキュリティグループ（SSH を許可）
resource "azurerm_network_security_group" "test_nsg" {
  name                = "test-nsg"
  location            = data.azurerm_resource_group.existing_rg.location
  resource_group_name = data.azurerm_resource_group.existing_rg.name

  security_rule {
    name                       = "SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_ranges    = ["22"]
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}
# NIC にセキュリティグループを関連付け
resource "azurerm_network_interface_security_group_association" "example" {
  network_interface_id      = azurerm_network_interface.test_nic.id
  network_security_group_id = azurerm_network_security_group.test_nsg.id
}


# Alma Linux 仮想マシン
resource "azurerm_linux_virtual_machine" "alma_vm" {
  name                = "alma-linux-vm"
  resource_group_name = data.azurerm_resource_group.existing_rg.name
  location            = data.azurerm_resource_group.existing_rg.location
  size                = "Standard_B1ls"  # VMのサイズ（任意で変更可能）
  admin_username      = "azureuser"        # 管理者ユーザー名

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS" # OSディスクのストレージタイプ
  }

  # SSH公開鍵を設定（ローカルのSSH公開鍵を指定）
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub") # ローカルに保存されたSSH公開鍵
  }

  network_interface_ids = [
    azurerm_network_interface.test_nic.id
  ]

  # OSイメージの指定（Alma Linux）
  source_image_reference {
    publisher = "almalinux"   # Alma LinuxのPublisher名
    offer     = "almalinux-x86_64"   # Offer名
    sku       = "8-gen2"     # SKU（バージョン）
    version   = "latest"
  }

  disable_password_authentication = true  # パスワード認証を無効にする

  # タグ設定（任意）
  tags = {
    environment = "production"
    owner       = "terraform"
  }
}
{% endhighlight %}


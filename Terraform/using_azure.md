1. Generate new keys directly on Desktop:

ssh-keygen -t rsa -b 4096 -f C:\Users\jjDok\Desktop\id_rsa

2. When SSH-ing into your VM later, you'll need to specify the key location:

ssh -i C:\Users\jjDok\Desktop\id_rsa addition@52.233.184.239


terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

provider "azurerm" {
  features {}
}

// Adding Resource Groups
resource "azurerm_resource_group" "addition" {
  name     = "eaddition-resources"
  location = "West Europe"
}

// Virtual Network
resource "azurerm_virtual_network" "addition" {
  name                = "addition-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.addition.location
  resource_group_name = azurerm_resource_group.addition.name
}

// Network Security Group
resource "azurerm_network_security_group" "addition" {
  name                = "addition-security-group"
  location            = azurerm_resource_group.addition.location
  resource_group_name = azurerm_resource_group.addition.name

  security_rule {
    name                       = "SSH"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = {
    environment = "Production"
  }
}

// Subnet
resource "azurerm_subnet" "addition" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.addition.name
  virtual_network_name = azurerm_virtual_network.addition.name
  address_prefixes     = ["10.0.2.0/24"]
}

// Connect NSG to Subnet
resource "azurerm_subnet_network_security_group_association" "addition" {
  subnet_id                 = azurerm_subnet.addition.id
  network_security_group_id = azurerm_network_security_group.addition.id
}

// Public IP address
resource "azurerm_public_ip" "addition" {
  name                = "addition-public-ip"
  location            = azurerm_resource_group.addition.location
  resource_group_name = azurerm_resource_group.addition.name
  allocation_method   = "Dynamic"
  sku                 = "Basic"
}

// Network Interface
resource "azurerm_network_interface" "addition" {
  name                = "addition-nic"
  location            = azurerm_resource_group.addition.location
  resource_group_name = azurerm_resource_group.addition.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.addition.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.addition.id
  }
}

// Virtual Machine
resource "azurerm_linux_virtual_machine" "addition" {
  name                = "addition-machine"
  resource_group_name = azurerm_resource_group.addition.name
  location            = azurerm_resource_group.addition.location
  size                = "Standard_F2"
  admin_username      = "addition"

  network_interface_ids = [
    azurerm_network_interface.addition.id,
  ]

  admin_ssh_key {
    username   = "addition"
    public_key = file("C:/Users/jjDok/Desktop/id_rsa.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
}

// Output
output "public_ip" {
  value = azurerm_public_ip.addition.ip_address
}
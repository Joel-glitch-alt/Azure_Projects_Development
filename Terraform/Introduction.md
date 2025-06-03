# What is Terraform?

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It allows you to define, provision, and manage cloud infrastructure and services using declarative configuration files. Terraform supports multiple cloud providers such as Azure, AWS, and Google Cloud, enabling consistent and repeatable infrastructure deployments.

**Key Features:**
- Declarative configuration language (HCL)
- Supports multi-cloud and hybrid environments
- Enables version control of infrastructure
- Automates resource provisioning and management


   ************* Terraform uses Three faces
   >>> init ==== Terraform initialises the project and identifies the provider to be used for the target environments.
   >>> plan === Terraform drafts a plan to get to the target phase.
   >>> apply === Terraform makes necessary changes required to mak it reflect on the target phase.

                               __________________________STRUCTURAL VIEW (Configuartion File)______________________________________

         resource "aws_instance" "webserver" {                //"aws_instance" = reource type /provider  //"webserver"=reource name 
            ami = "ami-0c2f25c1f66a1ff4d"                     // arguments.
            instance_type = "t2.micro"                        // arguments.
         }


                              _________________________TERRAFORM CONSISTS OF FOUR STEPS______________________________________
         1) Write the Configuration file. 
         2) Run the Terraform INIT command.
         3) Review the excusion plan using the terraform PLAN.
         4) Apply the changes using Terraform APPLY command.

               _______COMMON TERRAFORM COMMANDS________
         *** Terraform init
         *** terraform plan
         *** terraform apply
         *** terraform destroy.
         *** terraform refresh.
         *** terraform validate.
         *** terraform fmt. 
         *** terraform show.
         *** terraform providers.
         *** terraform output.
         *** terraform state list.


                             ___________________# Using a local set up!__________________________ 

                     terraform {
                           required_providers {
                           local = {
                           source = "hashicorp/local"
                        }
                     }
                  }

         provider "local" {}

            resource "local_sensitive_file" "pet" {
            content              = "This is a sensitive file"
            filename             = "${path.module}/files/outputfile"
            file_permission      = "0700"
            directory_permission = "0777"
         }
# This is a local file that will be created in the current directory


terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
}

provider "local" {}
provider "random" {}

resource "local_sensitive_file" "pet" {
  content              = "This is a sensitive file"
  filename             = "${path.module}/files/outputfile"
  file_permission      = "0700"
  directory_permission = "0777"
}

resource "local_file" "cat" {
  filename = "${path.module}/files/outputfile2"
  content  = "My favourite pet is called Penny Dollars"
}

resource "random_integer" "my_number" {
  min = 1
  max = 100
}

resource "random_pet" "random_name" {
  length    = 2
  separator = "-"
}

output "generated_number" {
  value = random_integer.my_number.result
}

output "generated_name" {
  value = random_pet.random_name.id
}

_________________________________________PROJECT TWO_________________________________________________
             ____________________________USING AZURE__(working)_____________________________

     # USING AZURE (creating a virtual machine & resouce groups)
     <!--  -->
provider "azurerm" {
  features {}
  subscription_id = "14430f9f-bd49-4721-b2ec-8b513c352e9a"
}

resource "azurerm_resource_group" "main" {
  name     = "firstResources"
  location = "westeurope"
}

resource "azurerm_virtual_network" "main" {
  name                = "first-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "main" {
  name                 = "first-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "main" {
  name                = "first-public-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Dynamic"
  sku                 = "Basic"
}

resource "azurerm_network_interface" "main" {
  name                = "first-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.main.id
  }
}

resource "azurerm_network_security_group" "main" {
  name                = "first-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Allow-HTTP"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "main" {
  subnet_id                 = azurerm_subnet.main.id
  network_security_group_id = azurerm_network_security_group.main.id
}

resource "azurerm_virtual_machine" "main" {
  name                  = "firstVM"
  location              = azurerm_resource_group.main.location
  resource_group_name   = azurerm_resource_group.main.name
  network_interface_ids = [azurerm_network_interface.main.id]
  vm_size               = "Standard_B1s"

  storage_os_disk {
    name              = "firstVM-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  storage_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }

  os_profile {
    computer_name  = "firstVm"
    admin_username = "addition"
    admin_password = "Youth@99"  # ⚠️ Not recommended for production use
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }

  tags = {
    environment = "TerraformDemo"
  }
}

***************************************************************************************************************************
Great! Since you have a static website (HTML, CSS, JavaScript) hosted on GitHub, you can easily run it on your Azure Ubuntu VM using Nginx — a lightweight web server.

Here’s a step-by-step guide to clone your project and serve it publicly:

****How to access your new VM
ssh <name of user>@<public-ip>


✅ Step-by-Step: Deploy Static Website on Azure VM

1. Install Nginx

sudo apt update
sudo apt install nginx -y

2. Clone your GitHub repository
Replace the URL with your GitHub repo:

cd /var/www
sudo git clone https://github.com/your-username/your-repo.git
Example:

sudo git clone https://github.com/Joel-glitch-alt/static-website.git

3. Configure Nginx
Let's assume your project is cloned to /var/www/static-website.

Edit the Nginx default config:

sudo nano /etc/nginx/sites-available/default
Replace the server block with:

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/static-website;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
4. Restart Nginx

sudo systemctl restart nginx
✅ Access Your Site
Open your browser and go to:

http://<your-public-ip>
Example:

http://20.71.183.45
✅ Tips
If you update your GitHub repo and want to pull changes:

cd /var/www/static-website
sudo git pull

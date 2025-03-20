How do i create a virtual machine in azure with an ssh and download jenkins  with the virtual machine

*1.2 Create a New Virtual Machine
Click "Create a resource" → Select "Virtual Machine".
Choose the subscription and resource group (or create a new one).
Enter a name for the VM (e.g., jenkins-vm).
Select the Region closest to your users.
Choose the Ubuntu 20.04 LTS as the Image (recommended for Jenkins).
Select Size (Standard B2s is good for small setups).
Under Administrator Account, select SSH public key.
Paste your public SSH key (or generate one if you don't have one).
Click "Next: Disks", select Standard SSD.
Click "Next: Networking", ensure:
Public IP is enabled.
Inbound port rules: Allow SSH (22) and HTTP (80).
Click Review + Create → Click Create.

***Step 2: Connect to the Virtual Machine via SSH
Copy the public IP address of your VM from Azure.
Connect via SSH:

ssh azureuser@<your-vm-public-ip>
Replace azureuser with your actual username.

***Step 3: Install Jenkins on Azure VM
3.1 Update and Install Java
Jenkins requires Java, so install it first:

sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-11-jdk

3.2 Add Jenkins Repository and Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

*3.3 Start and Enable Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

Step 5: Access Jenkins
Open a browser and go to:
http://<your-vm-public-ip>:8080


**To Get Jenkins Password
Run this command to get the password:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword


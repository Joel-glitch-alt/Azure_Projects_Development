1. Connect to your VM
On Linux/Ubuntu VM:
If you're on Windows, use an SSH client like PowerShell or PuTTY:

    ssh addition@<your-vm-public-ip>

 When using a virtual machine like(azure) for Jenkins & SonarQube. To be able to run SonarQube.
1. ✅ Install SonarScanner CLI on the VM
This is required so Jenkins can call sonar-scanner from the host system (not from inside a Docker container).

# Download and extract the latest SonarScanner CLI
For CMD use 
sudo unzip sonar-scanner-cli-5.0.1.3006-linux.zip -d /opt/


# Optional: Symlink for easier access
ln -s /opt/sonar-scanner-*/bin/sonar-scanner /usr/local/bin/sonar-scanner

# Confirm installation
sonar-scanner -v
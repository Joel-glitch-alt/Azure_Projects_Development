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



  USING JENKINS
***Jenkins is an open-source automation server that helps automate the parts of software development related to building, testing, and deploying, facilitating continuous integration and continuous delivery (CI/CD). It is highly extensible with a vast library of plugins that support building, deploying, and automating any project.

  Example use cases for Jenkins include:
***Automating the build and testing process for software projects.
***Implementing continuous integration and continuous delivery (CI/CD) pipelines.
***Deploying applications to various environments, such as development, staging, and production.
***Monitoring the execution of jobs and pipelines, and providing feedback to developers.


       JOB
***In Jenkins, a Job (also known as a Project) is a fundamental unit of work that Jenkins performs. A job defines a sequence of tasks that Jenkins will execute, such as building code, running tests, and deploying applications. Jobs can be configured to run on a schedule, in response to changes in source code, or triggered manually.

Key features of Jenkins Jobs include:
Build Triggers: Define when and how a job should be triggered, such as on a schedule, after another job completes, or when changes are detected in a version control system.
Build Steps: Specify the tasks that Jenkins should perform during the job, such as compiling code, running tests, and packaging applications.
Post-Build Actions: Define actions to be taken after the build steps are completed, such as archiving build artifacts, sending notifications, or deploying applications.
Parameterized Builds: Allow jobs to accept parameters, enabling dynamic and flexible builds based on user input or other conditions.
Pipeline Jobs: Use Jenkins Pipeline (defined in a Jenkinsfile) to create complex, multi-step workflows that can include parallel execution, conditional logic, and more.
Example use cases for Jenkins Jobs include:

Building Code: Compile source code from a version control system and generate build artifacts.
Running Tests: Execute unit tests, integration tests, and other automated tests to ensure code quality.
Deploying Applications: Deploy applications to various environments, such as development, staging, and production.
Automating Tasks: Perform routine tasks such as code analysis, documentation generation, and more.

      PIPELINE
In Jenkins, a Pipeline is a suite of plugins that supports implementing and integrating continuous delivery pipelines. A pipeline defines the entire workflow of your software development process, from building and testing to deploying and delivering applications. Pipelines are defined using a domain-specific language (DSL) called Groovy, and they can be version-controlled along with your source code using a Jenkinsfile.

======>>>>>>>>A simple example of a Jenkinsfile defining a pipeline:
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // Add build steps here
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Add test steps here
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Add deploy steps here
            }
        }
    }
}



   Example 2 of Pipelines
   pipeline {
    agent any
    stages {
        stage('Verify Docker') {
            steps {
                sh 'docker ps'
            }
        }
    }
}


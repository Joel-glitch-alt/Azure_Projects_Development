How do i create a virtual machine in azure with an ssh and download jenkins with the virtual machine

\*1.2 Create a New Virtual Machine
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
|
|
| \***\*\*\*\*\*\*\***\*\*\***\*\*\*\*\***\*\*\*\*\***\*\*\*\*\***\_\_\_\_\***\*\*\*\*\*\*\***\*\*\***\*\*\*\*\*\*\***\*\***\*\***\*\***\*\***
Inbound port rules: Allow SSH (22) and HTTP (80).
Click Review + Create → Click Create.

\*\*\*Step 2: Connect to the Virtual Machine via SSH
Copy the public IP address of your VM from Azure.
Connect via SSH:

open Bash or CMD...
ssh azureuser@<your-vm-public-ip>
Replace azureuser with your actual username.

Move to your Azure server >>> Network settings >> click on Create port rule >> change the 'Destination port range' to 8080 and 'priority' between 100 to 200.

\*\*\*Step 3: Install Jenkins on Azure VM
3.1 Update and Install Java
Jenkins requires Java, so install it first:

sudo apt update && sudo apt upgrade -y  
sudo apt install openjdk-17-jdk -y

3.2 Add Jenkins Repository and Install Jenkins

**\*\*\*\***\*\***\*\*\*\***For Debian-based systems (Ubuntu, Debian)...............

curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
 /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
 https://pkg.jenkins.io/debian binary/ | sudo tee \
 /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins

                  OPTION 2(for installing Jenkins)

\*\*\*\*\*\*\***\*\*\***\*\*\*\*\*\*\*** For RHEL-based systems (Red Hat, CentOS, Fedora, Rocky Linux, AlmaLinux) \***\*\*\*\***\*\*\*\***
.......................................................................................................................
sudo wget -O /etc/yum.repos.d/jenkins.repo \
 https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf upgrade

# Add required dependencies for the jenkins package

sudo dnf install fontconfig java-21-openjdk
sudo dnf install jenkins
sudo systemctl daemon-reload
........................................................................................................................

\*3.3 Start and Enable Jenkins

sudo systemctl enable jenkins
sudo systemctl start jenkins

\***\*\*\*\*\***\*\*\*\*\***\*\*\*\*\*** WHEN JENKINS HAS PROBLEM NOT WORKING.

🔧 Step 1: Check Jenkins Service Status
sudo systemctl status jenkins.service

Step 5: Access Jenkins
Open a browser and go to:
http://<your-vm-public-ip>:8080

\*\*To Get Jenkins Password
Run this command to get the password:

sudo cat /var/lib/<name-of-jenkins-project>/secrets/initialAdminPassword

USING SONARQUBE ON THE SAME SERVER OR VIRTUAL MACHINE

1.  sudo apt update

**\***\*\***\*\*\*\***\*\*** To install SonarQube using community standard**\*\*\*\***\*\*\*\***\*\*\*\*\*\*\*\*\*\*\*

...............................................................................................................................................

2.  wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.5.0.107428.zip?_gl=1*y4tn6k*_gcl_au*MTcwNjU2MjcwLjE3NDg5NDIxOTA.*_ga*NDI4MzIwMTE2LjE3NDg5NDIxOTA.*_ga_9JZ0GZ5TC6*czE3NDg5NDIxOTAkbzEkZzEkdDE3NDg5NDMyNDgkajYwJGwwJGgw

        > > Renane downloaded zip file
        > > mv 'sonarqube-25.5.0.107428.zip?\_gl=1*y4tn6k*\_gcl_au*MTcwNjU2MjcwLjE3NDg5NDIxOTA.*\_ga*NDI4MzIwMTE2LjE3NDg5NDIxOTA.*\_ga_9JZ0GZ5TC6\*czE3NDg5NDIxOTAkbzEkZzEkdDE3NDg5NDMyNDgkajYwJGwwJGgw' sonarqube-25.5.0.107428.zip

        *** Install unzip file
        >>> sudo apt install unzip -y

        *** Unzip file
        >>> unzip sonarqube-25.5.0.107428.zip

        >>> cd sonarqube-25.5.0.107428     >>> cd bin     >>> ls

        >>> cd ~/sonarqube-25.5.0.107428/bin/linux-x86-64 >> ./sonar.sh start >>http://<local>:9000

             Connecting to ip address

        >>> ip a

        ✅ 1. Update sonar.properties to listen on all interfaces

        >>> sudo nano ~/sonarqube-25.5.0.107428/conf/sonar.properties

    ...........................................................................................................................................
    OPTIONS 2 (REcommended)

3.  Install prerequisite packages:
    > > > sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
4.  Add Docker’s official GPG key:
    > > > curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
5.  Add Docker repository:

    > > > echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

6.  Update the package index again:

    > > > sudo apt update

7.  Install Docker Engine:

    > > > sudo apt install docker-ce docker-ce-cli containerd.io -y

8.  Start and enable Docker service:

    > > > sudo systemctl start docker
    > > > sudo systemctl enable docker

9.  Verify Docker installed correctly:

    > > > sudo docker --version

10. Run SonarQube on port 9500:
    sudo docker run -d \
    --name sonarqube \
    -p 9500:9000 \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    sonarqube:lts

11. SonarQube not connecting?, add your user to the docker group:
    Run the following command:
    sudo usermod -aG docker $USER

12. log out & log back in

    > > > newgrp docker

13. > > > docker ps

14. > > > ssh -i C:\Users\jjdok\Desktop\id_rsa joel@4.222.232.124

15. http://<your-server-ip>:9600

16.               ********************************************************************************************
                                          OPTION 3
                 ################### Create VM and Setup Jenkins, Azure CLI and Docker in It #############
            1 ) install Java in it
            $ sudo apt update
            $ sudo apt upgrade
            $ sudo apt install openjdk-17-jre
            $ java -version
            2 ) Install Jenkins
            Refer----https://www.jenkins.io/doc/book/installing/linux/
                     curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
                     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
                     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                     https://pkg.jenkins.io/debian binary/ | sudo tee \
                    /etc/apt/sources.list.d/jenkins.list > /dev/null

        > > > sudo apt-get update
        > > > sudo apt-get install jenkins

        >>>  $ sudo systemctl enable jenkins //Enable the Jenkins service to start at boot

        >>>  $ sudo systemctl start jenkins //Start Jenkins as a service
        >>>  $ systemctl status jenkins

3 ) Install Azure CLI
Refer --- https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

4 ) Install Docker >>> $ sudo apt-get install docker.io

> > > $ sudo usermod -aG docker $USER 
> > > $ sudo usermod -aG docker jenkins
> > > $ sudo init 6

################################# RELEASE TO AZURE WEB APPS #################################
az webapp config container set --name tetris-game-app --resource-group MyTest-RG --docker-custom-image-name mylabacr22.azurecr.io/tetris-game-app:latest

17. If Jenkins is running as a system service (e.g., on Ubuntu):
    🔄 Restart Jenkins:
    sudo systemctl restart jenkins

    ⏹ Stop Jenkins:
    sudo systemctl stop jenkins

    ▶ Start Jenkins:
    sudo systemctl start jenkins

    📋 Check status:
    sudo systemctl status jenkins

**\*\*\*\***\*\***\*\*\*\***\*\*\*\***\*\*\*\***\*\***\*\*\*\***\*\***\*\*\*\***\*\*\*\***\*\*\*\***\*\*\*\***\*\*\*\***\*\*\*\*\*\*

USING JENKINS
\*\*\*Jenkins is an open-source automation server that helps automate the parts of software development related to building, testing, and deploying, facilitating continuous integration and continuous delivery (CI/CD). It is highly extensible with a vast library of plugins that support building, deploying, and automating any project.

Example use cases for Jenkins include:
**_Automating the build and testing process for software projects.
_**Implementing continuous integration and continuous delivery (CI/CD) pipelines.
**_Deploying applications to various environments, such as development, staging, and production.
_**Monitoring the execution of jobs and pipelines, and providing feedback to developers.

       JOB

\*\*\*In Jenkins, a Job (also known as a Project) is a fundamental unit of work that Jenkins performs. A job defines a sequence of tasks that Jenkins will execute, such as building code, running tests, and deploying applications. Jobs can be configured to run on a schedule, in response to changes in source code, or triggered manually.

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
        stage('w/o docker') {
            steps {
                sh '''
                echo "without Docker"
                ls -la
                touch container-no.txt
                '''
            }
        }
        stage('w/ docker') {
            steps {
                sh '''
                echo "Running Node.js inside Docker"
                docker run --rm -v "$PWD:/workspace" -w /workspace node:18-alpine sh -c "ls -la && touch container-yes.txt"
                '''
            }
        }
    }

}

Example 2 of Pipelines (Correct and Working)

pipeline {
agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Ensure this matches the tool name in Jenkins Global Tool Configuration
        DOCKER_USERNAME = 'addition1905'
        DOCKER_IMAGE = 'addition1905/jenkins-nodejs:latest'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('SonarQube Analysis') {
             agent {
               docker {
                 image 'sonarsource/sonar-scanner-cli:latest'
                 reuseNode true
        }
    }
        steps {
             withSonarQubeEnv('sonar-server') {
                sh '''
                sonar-scanner \
                    -Dsonar.projectKey=my_project_key \
                    -Dsonar.projectName="My Project" \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src \
                    -Dsonar.language=js \
                    -Dsonar.sourceEncoding=UTF-8
            '''
        }
    }

}

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }

}

Using A JUnit Test Report
JUnit Test Report is a report generated by Jenkins (or other CI/CD tools) that provides detailed information about the execution of JUnit tests. JUnit is a popular testing framework for Java applications, and the test report helps developers understand the results of their tests, including which tests passed, which tests failed, and any errors or exceptions that occurred during the test execution.

pipeline {
agent any

    stages {
        stage('Build') {
            steps {
                // Compile and build your application
                sh './gradlew build'
            }
        }
        stage('Test') {
            steps {
                // Run JUnit tests
                sh './gradlew test'
            }
            post {
                always {
                    // Publish JUnit test results
                    junit 'build/test-results/test/*.xml'
                }
            }
        }
    }

}

MASTER OR CONTROLLER
In Jenkins, the Master (also known as the Controller) is the central server that manages the Jenkins environment. It is responsible for orchestrating the execution of jobs, managing build agents (also known as nodes or slaves), and providing the user interface for configuring and monitoring Jenkins.

Key responsibilities of the Jenkins Master include:

Job Scheduling: The master schedules and assigns jobs to build agents for execution.
Build Management: It manages the build process, including triggering builds, collecting build results, and storing build artifacts.
User Interface: The master provides a web-based user interface for configuring Jenkins, creating and managing jobs, and viewing build results and logs.
Plugin Management: It manages the installation and configuration of plugins, which extend Jenkins' functionality.
Security Management: The master handles user authentication, authorization, and security settings.

           AGENT

In Jenkins, an Agent (also known as a Node or Slave) is a machine that is configured to execute jobs assigned by the Jenkins Master (Controller). Agents can run on different operating systems and can be physical machines, virtual machines, or containers. They help distribute the workload, allowing Jenkins to run multiple jobs in parallel and improving the overall performance and scalability of the CI/CD pipeline.

RESTART JENKINS AND DOCKER
sudo systemctl restart jenkins
sudo systemctl restart docker

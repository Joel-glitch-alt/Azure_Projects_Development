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
1. Install Jenkins
2. Install all Tools needed like (Node.Js/ npm) on Jenkins agent.
3. Use Docker with Containers.

CONNECTING SONARQUBE AND DOCKER HUB TO JENKINS

1. Install SonarQube scanner plugins in Jenkins

2. Jenkins DashBoard >> Manage Jenkins >> Tools >> Click on 'Add SonanarQube Scanner', name it 'sonar-scanner', check mark 'install automatically' Click 'Save'.

\*\*\*\* Configure SonarQube Server Token.

3. Go to SonarQube Dashboard >> Click on 'Adminstration' >> Security >> Users >> then generate your TOKEN

   > > Provide the TOKEN name >> Expires in >> Click 'GENERATE'. copy token.

4. Manage Jenkins >> Security >> Credentials >> Click on 'global' >> Click on 'Add Credentials' >> select 'secret text'

   > > paste the SONARQUBE SECRET TEXT into the 'SECRET' bar >> provide ID (eg, sonar-token) and DESCRIPTION (eg, sonar-token), click "CREATE".

5. Navigate to your Docker Hub DashBaord, copy username (eg, addition1905).

6. Manage Jenkins >> Security >> Credentials >> Click on 'global' >> Click on 'Add Credentials' >> select (username and password), scope (Global...) >> provide docker hub username (eg, addition1905), >> provide your docker hub password >> ID (eg,dockerhub-credentials), same for description. click on 'CREATE'.

\*\*\* SETTING JENKINS SONARQUBE SERVER

7.  Manage Jenkins >> System >> scroll down to 'SonarQube servers' Click on 'Add SonarQube' >> Provide a name to be used in the Pipeline (eg,Jenkins-Sonar-Server), >> copy sonarqube url (eg, http://localhost:9000 no slash) and paste it in the 'server url', >> Server Authentication select the name of your sonarqube token (eg, sonar-token) >> click "Apply" and "SAVE".

8) SETTING UP GIT-HUB CREDENTIALS TO JENKINS

### 8) SETTING UP GITHUB CREDENTIALS IN JENKINS

- Go to your GitHub account, navigate to **Settings** > **Developer settings** > **Personal Access Tokens** > **Tokens (classic)**.
- Click **Generate new token (classic)**.
- Select the required scopes (check all except "delete repo"), then click **Generate token**.
- Copy the generated token.

- In Jenkins, go to **Manage Jenkins** > **Credentials** > **(global)** > **Add Credentials**.
- Choose **Username with password**.
- Enter your GitHub username and paste the generated token as the password.
- Provide an ID (e.g., `git-cred`) and a description, then click **Create**.

### 9) HOW TO ADD DOCKER HUB CREDENTIALS TO JENKINS

- Go to your Docker Hub account and copy your username.
- In Jenkins, go to **Manage Jenkins** > **Credentials** > **(global)** > **Add Credentials**.
- Select **Username with password**.
- Enter your Docker Hub username and password.
- Provide an ID (e.g., `dockerhub-credentials`) and a description, then click **Create**.

### 10) USING DOCKER IN YOUR JENKINS PIPELINE

- In your Jenkins pipeline, you can use the `docker` block to run stages inside Docker containers, or use `docker.build` and `docker.withRegistry` to build and push Docker images.
- Example snippet for building and pushing a Docker image:

  ```groovy
  stage('Docker Build & Push') {
      steps {
          script {
              def img = docker.build("${DOCKER_IMAGE}")
              docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                  img.push()
              }
          }
      }
  }
  ```

- Make sure your Jenkins agent has Docker installed and the necessary permissions to run Docker commands.

11. HOW TO ADD SONAQUBE QUALITY GATE TO JENKINS
    On the SonarQube UI >> Quality Gate >> Click on "Create" >> Provide a name eg. "sonar-quality-gate", >> Adminstration >> Click on Configuration >> Select Webhooks >> Click on "Create", >> Provide a name >> add your Jenkins URL >> Click on 'Create' to finish.

12. Move On To Create Your Pipeline

**\*\***\***\*\***Jenkins, Docker, SonarQube Integration source (https://www.youtube.com/watch?v=ScdedztTaAU).

\*\***\_\_\_\_**\*\***\*\***\*\***DEMO TO SHOW IF SONARQUBE UI AND JENKINS ARE CONNECTED**\*\***\*\***\*\***\_\_\_\_**\*\*\*\*\*\

pipeline {
agent any

    // Remove the tools section since we'll use the scanner directly
    // Remove custom environment variables since we'll use Jenkins configuration
    stages {
        stage('Checkout') {
            steps {
                echo 'Hello from Jenkins Pipeline!'
                // If you have a repo, uncomment the next line
                // checkout scm
            }
        }

        stage('Create Hello Project') {
            steps {
                script {
                    // Create a simple source file
                    writeFile file: 'hello.js', text: '''

// Simple Hello World function for SonarQube
function sayHello() {
console.log("Hello from SonarQube via Jenkins!");
return "Hello World";
}

function greetUser(name) {
if (name) {
console.log("Hello, " + name + "!");
} else {
console.log("Hello, World!");
}
}

// Example of code that might have quality issues for SonarQube to detect
function exampleFunction() {
var unused = "This variable is unused"; // Code smell
console.log("Hello SonarQube - analyzing code quality!");
}

// Call the functions
sayHello();
greetUser("SonarQube");
exampleFunction();
'''

                    // Create sonar-project.properties
                    writeFile file: 'sonar-project.properties', text: '''

# Required metadata

sonar.projectKey=jenkins-hello-world
sonar.projectName=Jenkins Hello World Project
sonar.projectVersion=1.0

# Source code location

sonar.sources=.
sonar.sourceEncoding=UTF-8

# Language settings

sonar.javascript.file.suffixes=.js

# Exclude Jenkins and SonarQube files

sonar.exclusions=**/\*.xml,**/\*.log,**/sonar-scanner/**
'''
}
}
}

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Hello! Starting SonarQube Analysis...'

                    // Use Jenkins configured SonarQube server
                    withSonarQubeEnv('sonar-server') {
                        def sonarScannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        sh """
                            ${sonarScannerHome}/bin/sonar-scanner \\
                                -Dsonar.projectKey=jenkins-hello-world \\
                                -Dsonar.projectName='Jenkins Hello World Project' \\
                                -Dsonar.projectVersion=1.0 \\
                                -Dsonar.sources=.
                        """
                    }

                    echo 'Hello! SonarQube Analysis completed!'
                }
            }
        }

        stage('Display Results') {
            steps {
                script {
                    echo 'Hello! Analysis has been sent to SonarQube!'
                    echo "Check your SonarQube dashboard at: http://68.154.50.4:9500"
                    echo 'Project: Jenkins Hello World Project'
                    echo 'Look for the Hello World JavaScript code in the analysis!'
                }
            }
        }
    }

    post {
        always {
            echo 'Hello! Pipeline completed.'
            echo "Check your SonarQube dashboard at: http://68.154.50.4:9500"
            echo 'Your Hello World project should now be visible in SonarQube!'
        }
        success {
            echo 'Hello! Pipeline succeeded! 🎉'
        }
        failure {
            echo 'Hello! Pipeline failed, but we tried our best! 😊'
        }
    }

}
//END

---

**\*\*\*\***\*\*\*\***\*\*\*\***\*\*\***\*\*\*\***\*\*\*\***\*\*\*\***COMPLETE PIPELINE FOR THE ABOVE PROJECT(SONARQUBE, DOCKERHUB) working.

pipeline {
agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner' // Matches Global Tool Config name
        DOCKER_USERNAME = 'addition1905'
        DOCKER_IMAGE = 'addition1905/jenkins-nodejs:latest'
    }

    stages {
        stage('Install & Build') {
            agent {
                docker {
                    image 'node:18'  // Full version includes npm and node
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "📦 Installing dependencies..."
                    [ -f package-lock.json ] && npm ci || npm install

                    echo "🏗️ Building app..."
                    npm run build || echo "No build script defined"

                    echo "✅ Build complete"
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "🧪 Running tests..."
                    npm test || echo "Tests failed or not defined"
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=my_project_key \
                          -Dsonar.projectName="My JS Project" \
                          -Dsonar.sources=./src \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                          -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

       stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }

}

        // Docker Build & Push and Deploy..
                    stage('Docker Build & Push') {
                    steps {
                    script {
                // 👇 Ensures Jenkins has access to Dockerfile and project files
                    checkout scm

                        def img = docker.build("${DOCKER_IMAGE}")
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                    img.push()
                }
            }
        }
        }

    }

    post {
        success {
            echo '✅ All steps completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }

}

**\*\***\*\*\*\***\*\***\*\*\*\***\*\*PROJECT TWO USING NODEJS (BACK END) working**\*\*\*\***\*\***\*\*\*\*\*\*\*\*\*\

pipeline {
agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_USERNAME = 'addition1905'
        DOCKER_IMAGE = 'addition1905/back-end-nodejs:latest'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Joel-glitch-alt/Best_Node_Express.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Jenkins-Sona-ServerTwo') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=Back_End_project_key \
                          -Dsonar.projectName="Back_End_NodeJS" \
                          -Dsonar.sources=./src \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                          -Dsonar.sourceEncoding=UTF-8
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def img = docker.build("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-key2') {
                        img.push()
                    }
                }
            }
        }
    }

}

    post {
        success {
            echo '✅ All steps completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }

**\*\***\*\*\*\***\*\***\_\_\***\*\*\*\*\***\*\*\***\*\*\*\*\***\*\*\*\***\*\***HOW TO RUN TEST IN PYTHON FILE**\*\*\*\***\*\***\*\*\*\*\***\*\*\***\*\*\*\*\***\_\_**\*\***\*\***\*\***-

1. Great question! To get test coverage of your Python code shown in SonarQube UI via a Jenkins pipeline, you need to:

2. Run your tests with coverage tracking (using coverage.py).

3. Generate a coverage report in XML format (coverage.xml).

4. Make sure SonarQube scans your project and imports the coverage report.

5. Configure Jenkins pipeline steps to do all this.

**\*\***\*\***\*\*** Step-by-step guide for your Jenkins pipeline:

1. Install dependencies
   Make sure you install:

coverage — to measure coverage.

pytest — or any other test runner.

Install SonarQube CLI when using vm

# Example (Ubuntu)

wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip
sudo mv sonar-scanner-5.0.1.3006-linux /opt/sonar-scanner

####

Click Add SonarQube Scanner

Name: sonar-scanner (exactly as used in your Jenkinsfile)

Uncheck “Install automatically”

Set the path to: /opt/sonar-scanner (or whatever path you used)

######

Example in your pipeline:

pip install coverage pytest 2. Run tests with coverage and generate XML report

**\*\***\*\*\***\*\***Run tests through coverage and output XML:
coverage run -m pytest
coverage xml -o coverage.xml
coverage run -m pytest runs tests collecting coverage data.

coverage xml generates coverage.xml, which SonarQube expects.

3. Configure SonarQube to read coverage.xml
   In your sonar-scanner command, specify:

bash
-Dsonar.python.coverage.reportPaths=coverage.xml

4. Sample Jenkins pipeline snippet
   groovy

pipeline {
agent any

    stages {
        stage('Install dependencies') {
            steps {
                sh 'pip install --upgrade pip coverage pytest'
            }
        }

        stage('Run Tests & Coverage') {
            steps {
                sh '''
                    coverage run -m pytest
                    coverage xml -o coverage.xml
                '''
                junit 'test-reports/results.xml'  // if you have junit tests reports
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('YourSonarQubeServer') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=your_project_key \
                          -Dsonar.sources=. \
                          -Dsonar.python.coverage.reportPaths=coverage.xml \
                          -Dsonar.python.xunit.reportPath=test-reports/results.xml
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
    }

}

Notes:
You can generate test result reports with pytest and convert them to JUnit XML (--junitxml=test-reports/results.xml) if you want to show test results in SonarQube.

Make sure coverage.xml is in the directory where sonar-scanner runs or provide full path.

Your sonar-project.properties or sonar-scanner CLI options must point to the correct paths.

waitForQualityGate Jenkins step waits for SonarQube's analysis to finish and returns status.

In summary:
Use coverage run to collect coverage during tests.

Use coverage xml to generate coverage.xml.

Pass that file path to SonarQube (sonar.python.coverage.reportPaths).

Run SonarQube scanner in Jenkins pipeline.

Use waitForQualityGate to get result in Jenkins.

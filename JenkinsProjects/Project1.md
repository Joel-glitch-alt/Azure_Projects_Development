1) Install Jenkins
2) Install all Tools needed like (Node.Js/ npm) on Jenkins agent.
3) Use Docker with Containers.


CONNECTING SONARQUBE AND DOCKER HUB TO JENKINS
1) Install SonarQube scanner plugins in Jenkins 

2) Jenkins DashBoard >> Manage Jenkins >> Tools >> Click on 'Add SonanarQube Scanner', name it 'sonar-scanner', check mark 'install automatically' Click 'Save'.

**** Configure SonarQube Server Token.

3) Go to SonarQube Dashboard >> Click on 'Adminstration'  >> Security >> Users >> then generate your TOKEN
 >> Provide the TOKEN name >> Expires in >> Click 'GENERATE'. copy token.

4) Manage Jenkins >> Security >> Credentials >> Click on 'global' >> Click on 'Add Credentials' >> select 'secret text'
>> paste the SONARQUBE SECRET TEXT into the 'SECRET' bar >> provide ID (eg, sonar-token) and DESCRIPTION (eg, sonar-token), click "CREATE".

5) Navigate to your Docker Hub DashBaord, copy username (eg, addition1905).

6) Manage Jenkins >> Security >> Credentials >> Click on 'global' >> Click on 'Add Credentials'  >> select (username and password), scope (Global...) >> provide docker hub username (eg, addition1905), >> provide your docker hub password >> ID (eg,dockerhub-credentials), same for description. click on 'CREATE'.

*** SETTING JENKINS SONARQUBE SERVER
7) Manage Jenkins >> System >> scroll down to 'SonarQube servers' Click on 'Add SonarQube' >> Provide a name to be used in the Pipeline (eg,Jenkins-Sonar-Server), >> copy sonarqube url (eg, http://localhost:9000 no slash) and paste it in the 'server url', >> Server Authentication select the name of your sonarqube token (eg, sonar-token) >> click "Apply" and "SAVE".


8) SETTING UP GIT-HUB CREDENTIALS TO JENKINS
 Go to repositories and navigate to settings >> Developer settings >> Personal Access Tokens >> Tokens(classic) >> Generate New Token >>Generate new token classic >> Check Mark all except delete repo >> click on generate token.

 9) Move to Jenkins >> Manage Jenkins >> Credentials >> Global >> 'username and password', provide 'Git-Hub username' and the Token generated as password.

 10) HOW TO ADD DOCKER-HUB.
  

11) Move On To Create Your Pipeline

....Jenkins, Docker, SonarQube Integration source (https://www.youtube.com/watch?v=ScdedztTaAU).


   COMPLETE PIPELINE FOR THE ABOVE PROJECT(SONARQUBE, DOCKERHUB) working.
***************************************************************************************************************************
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


************************************PROJECT TWO USING NODEJS (BACK END) working****************************************
JENKINS PIPELINE

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


__________________________________****************HOW TO RUN TEST IN PYTHON FILE*****************______________________________-

1) Great question! To get test coverage of your Python code shown in SonarQube UI via a Jenkins pipeline, you need to:

2) Run your tests with coverage tracking (using coverage.py).

3) Generate a coverage report in XML format (coverage.xml).

4) Make sure SonarQube scans your project and imports the coverage report.

5) Configure Jenkins pipeline steps to do all this.

************** Step-by-step guide for your Jenkins pipeline:
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

pip install coverage pytest
2. Run tests with coverage and generate XML report 

***************Run tests through coverage and output XML:
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


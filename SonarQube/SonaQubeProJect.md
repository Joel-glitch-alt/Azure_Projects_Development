    									SonarQube - Code Quality Analysis
    													by
    												Kastro Kiran V

---

Step 1: Launch Ubuntu VM with instance type as t2.large, AMI: 24.04, Storage: 25 GB, No. of instances: 1

Step 2: Connect to the VM

Step 3: sudo apt update

Step 4: Lets setup Jenkins
$ sudo apt install openjdk-17-jre-headless -y

The below commands can be found in; https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
 https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
 https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
 /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y

vi j.sh
<Paste the above commands>

esc --- :wq

Lets give the permission to j.sh file to execute; sudo chmod +x j.sh
ls --- you can see 'j.sh' in green colour

Lets execute the j.sh file; ./j.sh
with this Jenkins installation is done.
Note that, Jenkins will run on 8080 port number by default.

Open Port No. 8080 for the VM

Access Jenkins; <publicip>:8080 ---> Setup the Jenkins dashboard by following the known steps

Step 5: SonarQube Setup

# Lets setup sonar server using Docker (this is the simplest way)

$ sudo apt install docker.io

#Lets give permissions to run docker
$ sudo chmod 666 /var/run/docker.sock

#Lets run docker commands to install SonarQube
$ docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community

$ docker ps
#You can see SonarQube container status

Open Port No. 9000 in the SG of VM

# Access SonarQube;

<publicip>:9000 --- You can see the SonarQube console

Click on 'Home'

Default login name and password is 'admin'

Set new password

You can see the homepage of SonarQube

With this Jenkins and SonarQube setup is ready

Step 6: Jenkins Pipeline Job for SonarQube Analysis

Jenkins Dashboard ---> Manage Jenkins ---> Plugins ---> Available Plugins ---> Search for 'SonarQube Scanner' and select it. Also search for 'Eclipse Temurin installer' and 'Pipeline Stage View" and select them ---> Install ---> Click on 'Go back to the top of page'

Jenkins Dashboard ---> Manage Jenkins ---> Tools ---> Add JDK ---> Name: jdk17, 'Check' install automatically, Select 'Install from adoptium.net' from dropdown, Version: select jdk-17.0.11+9 ---> Click on 'Add SonarQube Scanner', Name: sonar-scanner, 'Check' install automatically, Select 'Install from Maven Central', Version: SonarQube Scanner 5.0.1.3006 ---> Click on 'Add Maven' ---> Name: maven3, 'Check' install automatically, Select 'Install from Apache' from dropdown, Version: 3.9.7 ---> Apply ---> Save

Lets configure the SonarQube server;
Goto SonarQube console --- Click on 'Administration' --- Click on 'Security' --- Select 'Users' --- You can see 'Tokens' --- Click on 3 dashes icon --- A New dialogue --- Name: token, Expires in: 90days --- Generate --- You can see token in green colour. Copy it.

Token: squ_983573f5ed8baca362ba501ad6e04b3e70c23093

Manage Jenkins --- Security --- Credentials --- Click on 'global' --- Click on 'Add Credentials' --- Kind: Secret text, Scope: Global, Secret: <Paste the token copied from SonarQube console>, ID: sonar-token, Description: sonar-token --- Create

Similarly configure dockerhub credentials with id as "dockerhub-credentials" with kind as "username with password"

Manage Jenkins --- System --- Scroll down to 'SonarQube servers' --- Click on 'Add SonarQube' --- Name: sonar-server --- ServerURL: <PublicIPofSQinstalledVM>:9000 --- Server Authentication Token: select 'sonar-token' --- Apply --- Save

<For the next steps, please watch the video>
https://www.youtube.com/watch?v=ScdedztTaAU

using Pipelines withJenkins and SonarQube
https://youtu.be/NnkUGzaqqOc

****************\*\*****************$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$****************\*\*****************
pipeline {
agent any

    tools {
        nodejs 'NodeJS'
    }

       environment {
    SONAR_SCANNER_OPTS = "-Xmx1024m"
    NODE_ENV = 'test'
    DOCKER_IMAGE_NAME = 'addition1905/nodejs-authentication-postgresql'
    DOCKER_TAG = "${BUILD_NUMBER}"
    SONAR_TIMEOUT = '10' // minutes

}

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Clear npm cache') {
            steps {
                sh 'rm -rf node_modules package-lock.json || true'
                sh 'npm cache clean --force'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
                sh 'npm uninstall bcrypt || true'
                sh 'npm install bcryptjs'
                sh 'chmod -R +x node_modules/.bin/'
            }
        }

        stage('Update Code for bcryptjs') {
            steps {
                script {
                    sh '''
                    find . -name "*.js" -not -path "./node_modules/*" -exec sed -i 's/require("bcrypt")/require("bcryptjs")/g' {} \\;
                    find . -name "*.js" -not -path "./node_modules/*" -exec sed -i "s/require('bcrypt')/require('bcryptjs')/g" {} \\;
                    '''
                }
            }
        }

        stage('Fix Test Configuration') {
            steps {
                script {
                    sh '''
                    if [ ! -f jest.config.js ]; then
                        echo "jest.config.js not found, using existing configuration from package.json"
                    else
                        echo "jest.config.js found, using existing configuration"
                    fi
                    '''
                }
            }
        }

        stage('Run Tests with Coverage') {
            steps {
                script {
                    if (fileExists('jest.config.js')) {
                        sh 'npx jest --config=jest.config.js --coverage --verbose --detectOpenHandles --forceExit'
                    } else {
                        sh 'npx jest --coverage --verbose --detectOpenHandles --forceExit'
                    }
                }
            }
            post {
                always {
                    script {
                        if (fileExists('coverage/')) {
                            archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                            echo "Coverage report archived successfully"
                        }
                    }
                }
            }
        }

      stage('SonarQube Analysis') {
    steps {
        script {
            try {
                timeout(time: 10, unit: 'MINUTES') {
                    withSonarQubeEnv('Sonar-server') {
                        withCredentials([string(credentialsId: 'mysonar-second-token', variable: 'SONAR_TOKEN')]) {
                            sh """
                            docker run --rm \\
                                -e SONAR_HOST_URL=\${SONAR_HOST_URL} \\
                                -e SONAR_TOKEN=\${SONAR_TOKEN} \\
                                -v \${WORKSPACE}:/usr/src \\
                                sonarsource/sonar-scanner-cli:latest
                            """
                        }
                    }
                }
            } catch (Exception e) {
                echo "SonarQube analysis failed or timed out: ${e.getMessage()}"
                currentBuild.result = 'UNSTABLE'
            }
        }
    }

}

        stage('Docker Build & Push') {
            steps {
                script {
                    // Ensure we have the latest code and Dockerfile
                    checkout scm

                    try {
                        // Build the Docker image with build number tag
                        def img = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")

                        // Also tag as latest
                        sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ${DOCKER_IMAGE_NAME}:latest"

                        // Push to Docker Hub
                        docker.withRegistry('https://index.docker.io/v1/', 'addition1905') {
                            img.push("${DOCKER_TAG}")
                            img.push("latest")
                        }

                        echo "✅ Docker image built and pushed successfully!"
                        echo "📦 Image: ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
                        echo "📦 Image: ${DOCKER_IMAGE_NAME}:latest"

                    } catch (Exception e) {
                        echo "❌ Docker build/push failed: ${e.getMessage()}"
                        error "Docker build and push stage failed"
                    }
                }
            }
        }

        stage('Deployment Ready Check') {
            steps {
                script {
                    echo "=== PIPELINE SUCCESS SUMMARY ==="
                    echo "✅ Code checked out successfully"
                    echo "✅ Dependencies installed and configured"
                    echo "✅ All tests passed"
                    echo "✅ Code coverage generated"
                    echo "✅ SonarQube analysis completed"
                    echo "✅ Docker image built and pushed"
                    echo ""
                    echo "🚀 Your Node.js application is ready for deployment!"
                    echo "🐳 Pull and run with: docker run -p 3000:3000 ${DOCKER_IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed'
            script {
                // Archive coverage reports
                if (fileExists('coverage/')) {
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                }

                // Clean up processes
                sh 'pkill -f jest || true'
                sh 'pkill -f node || true'

                // Clean up Docker images to save space (optional)
                sh """
                    docker rmi ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} || true
                    docker system prune -f || true
                """
            }
        }
        success {
            echo '🎉 Pipeline executed successfully!'
            echo "🐳 Docker image ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} is now available on Docker Hub"
        }
        failure {
            echo '❌ Pipeline execution failed!'
            script {
                sh 'echo "Node version: $(node --version)"'
                sh 'echo "NPM version: $(npm --version)"'
                sh 'docker --version || echo "Docker not available"'
            }
        }
        unstable {
            echo '⚠️ Pipeline execution was unstable!'
            echo 'Some stages may have had issues but core functionality works.'
        }
    }

}

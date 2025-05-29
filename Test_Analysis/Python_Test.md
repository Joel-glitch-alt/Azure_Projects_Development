
Perfect! Since you’re using Jenkins with SonarQube to analyze a Python project, here's a complete step-by-step guide from scratch to full automation:

✅ OVERVIEW OF THE PIPELINE

Your Jenkins pipeline will do the following:

Checkout code from GitHub.

Install dependencies.

Run unit tests with coverage.

Generate coverage.xml.

Run sonar-scanner to analyze the project.

Wait for SonarQube Quality Gate.

                 🧱 PROJECT STRUCTURE (Example)

python-sonar-test/
├── calc.py
├── tests/
│   └── test_calc.py
├── requirements.txt
├── coverage.xml         # Generated
├── sonar-project.properties
└── Jenkinsfile

    🧰 PREREQUISITES
On Jenkins:
✅ Install the SonarQube Scanner plugin.

✅ Install the Pipeline and Git plugins.

✅ Configure SonarQube server:

Manage Jenkins → Configure System

Scroll to SonarQube servers

Add your server name (e.g., Jenkins-sonar-server) and token (generated in SonarQube).

✅ Configure SonarScanner in Global Tool Configuration

✅ Add Python and pip to your Jenkins agent (can be a system path or tool).

    🧪 requirements.txt

pytest
coverage

   📄 sonar-project.properties
sonar.projectKey=python-sonar-test
sonar.projectName=Python Sonar Test
sonar.projectVersion=1.0

sonar.sources=.
sonar.language=py
sonar.sourceEncoding=UTF-8
sonar.python.coverage.reportPaths=coverage.xml


        📜 Jenkinsfile
        
pipeline {
    agent any

    tools {
        // If defined in Jenkins -> Global Tool Configuration
        // python "Python3" or leave this block if system python is used
    }

    environment {
        PATH = "${tool 'sonar-scanner'}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests & Generate Coverage') {
            steps {
                sh '''
                    coverage run -m pytest
                    coverage xml
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Jenkins-sonar-server') {
                    sh 'sonar-scanner'
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
    }

    post {
        always {
            echo '🔎 Pipeline completed. Check SonarQube dashboard.'
        }
        failure {
            echo '❌ Pipeline failed. See logs.'
        }
    }
}


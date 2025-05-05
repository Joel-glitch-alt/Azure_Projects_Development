1) to run Jenkins Container on Docker 
  >>> docker run -p 8080:8080 jenkins/jenkins
2) The password shows in the terminal eg "a5115916e153414ba0fb2ba24e3bdebe" copy paste in Unlock Jenkins in Adminstrative password.
3) Install required plugins


4) Examples of Pipeline using Node
pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "without Docker"'
            }
        }

        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh 'node -v'
            }
        }
    }
}



OPTION 2
✅ Goal:
You want to run Jenkins in Docker with Docker CLI support so it can pull images like node:18-alpine during pipeline execution.

🔧 Step 1: Create a Jenkins Dockerfile with Docker CLI
Create a folder named jenkins-docker:

mkdir jenkins-docker
cd jenkins-docker
Create a Dockerfile inside this folder:

Dockerfile

FROM jenkins/jenkins:lts

USER root

# Install Docker CLI
RUN apt-get update && \
    apt-get install -y docker.io && \
    usermod -aG docker jenkins

USER jenkins
This adds Docker CLI to Jenkins and allows the jenkins user to run it.

🔨 Step 2: Build the Docker image
Run this in the same folder where the Dockerfile is:


docker build -t jenkins-docker .
🚀 Step 3: Run Jenkins with access to Docker socket

docker run -d --name jenkins-docker \
  -p 8080:8080 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-docker
-v /var/run/docker.sock:/var/run/docker.sock lets Jenkins control the host Docker engine.

jenkins_home is the volume for persistent Jenkins data.

🌐 Step 4: Open Jenkins in your browser
Go to:

http://localhost:8080
🔑 Step 5: Unlock Jenkins
Get the password:


docker exec jenkins-docker cat /var/jenkins_home/secrets/initialAdminPassword
Use this password to unlock Jenkins and complete setup.

🔌 Step 6: Install Required Plugins
Install:

Docker Pipeline

Pipeline

Blue Ocean (optional)

📦 Step 7: Create a sample Jenkins Pipeline
Example Jenkinsfile:

pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "without Docker"'
            }
        }
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh 'node --version'
            }
        }
    }
}


docker run -d ^
  --name jenkins ^
  -p 8080:8080 ^
  -p 50000:50000 ^
  -v jenkins_home:/var/jenkins_home ^
  -v //./pipe/docker_engine:/var/run/docker.sock ^
  -e DOCKER_HOST=unix:///var/run/docker.sock ^
  --group-add 999 ^
  jenkins/jenkins:lts



After applying any method, test with:

docker exec jenkins docker run --rm hello-world

Or Better Yet - Use the Official Jenkins Docker Image with Sidecar
Create a docker-compose.yml file:

yaml
version: '3'
services:
  jenkins:
    image: jenkins/jenkins:lts
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    depends_on:
      - dind
  dind:
    image: docker:dind
    privileged: true
    volumes:
      - docker_data:/var/lib/docker
volumes:
  jenkins_home:
  docker_data:
Run it with:

cmd
docker-compose up -d
Why This Works Better:
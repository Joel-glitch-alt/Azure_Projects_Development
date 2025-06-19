HOW TO RUN JENKINS AND SONARQUBE ON DOCKER CONTAINER USING UBUNTU

---

STEP 1
RUNNING JENKINS ON DOCKER CONTAINER SERVERLESS
**\*\***\***\*\***Using Serverless Docker

sudo apt update

docker run -d \
--name joel-jenkins \
-p 8080:8080 -p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v jenkins_home:/var/jenkins_home \
-u root \
jenkins/jenkins:lts

example 2 (second Jenkins-Docker container fresh project).

sudo apt update

docker run -d \
--name addition-jenkins-fresh \
-p 8081:8080 \
-p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v jenkins_home_fresh:/var/jenkins_home \
-u root \
jenkins/jenkins:lts

\*\*\*To get password of fresh container
docker exec <name-of-jenkins-name> cat /var/jenkins_home/secrets/initialAdminPassword

\***\*\*\*\*\*\*** Get IP address of your serverless host
ip a

🛠️ Option 2: Install Docker CLI inside Jenkins container

docker exec -it <name of conatiner> bash

Then inside the container:
apt-get update
apt-get install -y docker.io
After this, try: docker --version inside the container.

---

STEP 2 2) TO GET THE JENKINS PASSWORD

****\*\*\*\*****\*\*\*\*****\*\*\*\*****To get the initial admin password for Jenkins and complete the setup, run:
docker exec -it <name of container> cat /var/jenkins_home/secrets/initialAdminPassword

---

STEP 3 3) TO GET SONARQUBE CONTAINER ON A SERVERLESS HOST

USING SONARQUBE SERVER

docker run -d \
--name sonarqube \
-p 9000:9000 \
-v sonarqube_data:/opt/sonarqube/data \
-v sonarqube_extensions:/opt/sonarqube/extensions \
-v sonarqube_logs:/opt/sonarqube/logs \
sonarqube:lts

---

\*To start your Container
docker start jenkins-dind-new

\*To restart your Docker Container
docker restart jenkins-dind-new

*Access Jenkins inside the container:
*After the container is started, you can access it using:
docker exec -it jenkins-dind-new bash

\*To start my SonarQube Container
sudo docker start SonarQube

---

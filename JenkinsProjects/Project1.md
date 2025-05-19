1) Install Jenkins
2) Install all Tools needed like (Node.Js/ npm) on Jenkins agent.
3) Use Docker with Containers.


CONNECTING SONARQUBE AND DOCKER HUB TO JENKINS
1) Install SonarQube scanner plugins in Jenkins 
2) Jenkins DashBoard >> Manage Jenkins >> Tools >> Click on 'Add SonanarQube Scanner', name it 'sonar-scanner', check mark 'install automatically' Click 'Save'.
**** Configure SonarQube Server
3) Go to SonarQube Dashboard >> Click on 'Adminstartion'  >> Security >> Users >> then generate your TOKEN
 >> Provide the TOKEN name >> Expires in >> Click 'GENERATE'. copy token.

4) Manage Jenkins >> Security >> Credentials >> Click on 'global' >> Click on 'Add Credentials' >> select 'secret text'
>> paste the SONARQUBE SECRET TEXT into the 'SECRET' bar >> procide ID (eg, sonar-token) and DESCRIPTION (eg, sonar-token), click "CREATE".

5) Navigate to your Docker Hub DashBaord, copy username (eg, addition1905).
6) Manage Jenkins >> Security >> Credentials >> Click on 'global' >> Click on 'Add Credentials'  >> select (username and password), scope (Global...) >> provide docker hub username (eg, addition1905), >> provide your docker hub password >> ID (eg,dockerhub-credentials), same for description. click on 'CREATE'.

*** SETTING JENKINS SONARQUBE SERVER
7) Manage Jenkins >> System >> scroll down to 'SonarQube servers' Click on 'Add SonarQube' >> Provide a name to be used in the Pipeline (eg,Jenkins-Sonar-Server), >> copy sonarqube url (eg, http://localhost:9000 no slash) and paste it in the 'server url', >> Server Authentication select the name of your sonarqube token (eg, sonar-token) >> click "Apply" and "SAVE".

8) Move On To Create Your Pipeline


....Jenkins, Docker, SonarQube Integration source (https://www.youtube.com/watch?v=ScdedztTaAU).
# SonarQube and Nexus installation and Jenkins integration

## Install docker as we will run SonarQube and Nexus containers

Add Docker's official GPG key

```bash	
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

	# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
 ```
then install using this command
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
By default all the users do not have permissions to run the docker commands thus, run the below command
```bash
sudo chmod 666 /var/run/docker.sock
```
this is not the recommended approach a better approach would be to create a docker and then add user to it.

---
## To install and run nexus will download the docker image and use the default port 8081
```bash
docker run -d --name nexus -p 8081:8081 sonatype/nexus3:3.66.0
```
```admin``` is the default username for password we need to 
```bash
docker exec -it <container-it> /bin/bash
```
- and cat using the given path
- to access the container use <vm's-publicip>:<exposed-port>

## Installing SonarQube container use default port 9000
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
default username ```admin``` and password ```admin```

---

## Jenkins SonarQube integration / SonarQube Analysis
	
For SonarQube analysis first set up the server in Jenkins
- Add the secret text/ token in Jenkins global credentials
- SonarQube-> Administration-> security-> users -> three bars -> give a name and generate one
- then in Jenkins credentials, choose secret text as option, give id.
- configure the SonarQube in Jenkins by going into system -> SonarQube server, give name, SonarQube server Ip address with port and without the last /, choose the credential we just added and save
		
```yaml
		    environment {
        SCANNER_HOME = tool 'sonar-scanner' //as configured in the Jenkins tools 
			}
```
```yaml		
		stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') { //simple writing 'sonar' because we just configured it in system
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                    -Dsonar.java.binaries=. '''    
            }
        }
```
## SonarQube Quaity Gate, webhooks

Generate a webhook
	Administration- configuration- webhook- create
- Give a name
- for the URL this is the format
```<http://jenkins-public-ip:8080>/sonarqube-webhook/``` then save
				
- in pipeline use this script
```yaml	
		stage('Quality Gate') { // code quality check by setting webhook in SonarQube
            steps {
                script  {
                    waitForQualityGate abortPipeline: false, credentialsID: '<credential ID>' //sonar token
                }
            }
        }
```
## Publishing artifacts to nexus 

for publishing the artifacts to nexus we need to edit the pom.xml file in github with the below section
```xml
	<distributionManagement>
		<repository>
			<id>maven-releases</id>
			<url>http://54.167.177.28:8081/repository/maven-releases/</url> // get the url from nexus - browse - maven releases
		</repository>
		<snapshotRepository>
			<id>maven-snapshots</id>
			<url>http://54.167.177.28:8081/repository/maven-snapshots/</url>
		</snapshotRepository>
	</distributionManagement>
```
			
	-> setting up credentials to access the nexus repo 
		manage jenkins - managed files // we get this option cause we installed a plugin 'Config File Provider'
		
		-> inside contents we are going to edit some information
			under servers we are gonna add both maven-releses and maven-snapshots like this
				-->
				<server>
				  <id>maven-releases</id>
				  <username>admin</username>	//password and username used for nexus
				  <password>sanjay</password>
				</server>

				 <server>
				  <id>maven-snapshots</id>
				  <username>admin</username>
				  <password>sanjay</password>
				</server>
				-->
			and save
		
	-> in pipeline script
		-> pipeline syntac withMaven: Provide Maven Enviroment

			stage('Publish to nexus') {
				steps {
					withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
						sh "mvn deploy"
					}
				}
			}
			


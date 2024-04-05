# Jenkins installation and configuration with necessary plugins

search for the official page for commands for installation second part

```bash
	sudo apt install openjdk-17-jre-headless 
	
	sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
	  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
	echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
	  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
	  /etc/apt/sources.list.d/jenkins.list > /dev/null
	sudo apt-get update
	sudo apt-get install jenkins
```
	
for the devops project we also need to install docker here
	
	
## These are some of the plugins used
```	
  Eclipse Termurin installer      - to setup multiple versions of JDK
  Config File Provider            - to create global setting file for nexus
  Pipeline Maven Integration
  Maven Integration
  SonarQube Scanner               - this is the tool to do the analysis and the report will be published to server
  Docker
  Docker pipeline
  Kubernetes
  Kubernetes CLI
  Kubernetes Client API
  Kubernetes Credentials
```
	

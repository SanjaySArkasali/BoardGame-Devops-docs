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


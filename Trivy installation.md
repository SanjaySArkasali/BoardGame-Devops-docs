# Trivy installation and configuration 

## Trivy is a open source tool for scanning the dependency 

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```
	
## Trivy file system scanning
```bash
trivy fs --foramt table -o trivy-fs-report.html .
```

```
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner' //configured in the jenkins tools 
    }

    stages {
        stage('git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/SanjaySArkasali/Devops-project-1.git'
                }
            }
    
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('File system scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') { //simple writing 'sonar' because we just configured it in system
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                    -Dsonar.java.binaries=. '''    
                }
            }
        }
        
        stage('Quality Gate') { // code quality check by setting webhook in sonarQube
            steps {
                script  {
                    waitForQualityGate abortPipeline: false, credentialsID: 'sonar-token'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Publish to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        
        stage('Build and Tag Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker build -t sanark/boardgame:latest .'
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html sanark/boardgame:latest'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker push sanark/boardgame:latest'
                    }
                }
            }
        }
        
        stage('kubernetes deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.87.194:6443') {
                    sh 'kubectl apply -f deployment-service.yaml'
                }
            }
        }
        
        stage('verify deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.87.194:6443') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }

post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'sanjaysarkasali@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
              )
          }
      }
  }
}
```

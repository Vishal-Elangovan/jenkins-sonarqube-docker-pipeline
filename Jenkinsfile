pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://43.205.207.247:9000'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Vishal-Elangovan/jenkins-sonarqube-docker-pipeline.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner/bin/sonar-scanner \
                          -Dsonar.host.url=$SONARQUBE_URL \
                          -Dsonar.login=$SONAR_AUTH_TOKEN \
                          -Dsonar.projectKey=pipeline-website \
                          -Dsonar.projectBaseDir=.
                    '''
                }
            }
        }

        stage('Copy files to Docker Server') {
            steps {
                sshagent(['docker-ec2-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r \
                        404.html "ABOUT THIS TEMPLATE.txt" Dockerfile Jenkinsfile README.md \
                        contact.html css fonts images index.html js login.html password-reset.html \
                        register.html test.txt videos \
                        ubuntu@3.108.227.128:~/website/
                    '''
                }
            }
        }

        stage('Deploy on Docker Server') {
            steps {
                sshagent(['docker-ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@3.108.227.128 << EOF
                        cd ~/website
                        docker build -t website:latest .
                        docker stop website || true
                        docker rm website || true
                        docker run -d --name website -p 8081:80 website:latest
                        EOF
                    '''
                }
            }
        }
    }
}

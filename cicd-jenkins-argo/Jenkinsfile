pipeline {
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonarqube'
    }
    stages {
        stage('GIT CHECKOUT') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'giti_cred', poll: false, url: 'https://github.com/schets14/DevOps.git'
            }
        }
        stage('COMPILE CODE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASP SCANNING') {
            steps {
                  dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                  dependencyCheckPublisher pattern:  '**/dependency-check-report.xml'
            }
        }
        stage('SONARQUBE SCANNING ANALYSIS') {
            steps {
                  withSonarQubeEnv('sonarqube-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Ekart '''
                  }
            }
        }
        stage('BUILD CODE') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('DOCKER BUILD & PUSH') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t schets14/chetan:latest -f docker/Dockerfile ."
                    sh "docker push schets14/chetan:latest "
                        
                    }
                    
                }
            }
        }
        stage('DEPLOY') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker run -d  -p 8070:8070 --name=shoping-app schets14/chetan:latest"
                        
                    }
                    
                }
            }
        }
    }
}

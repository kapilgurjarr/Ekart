pipeline {
    agent any
    tools {
        jdk 'jdk-17'
        maven 'maven-3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/kapilgurjarr/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://35.154.143.173:9000/ -Dsonar.login=squ_84e01371e54a28a24e514f87f5e4d757f24a557a -Dsonar.projectName=shopping-cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=shopping-cart
                    '''
            }
        }
        
        stage('OWASP Scan') {
            steps {
                script {
                    try {
                        dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        
        stage('Build Application') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '58df9d33-95db-4f0a-bd59-52954ae53708', toolName: 'docker') {
                        sh 'docker build -t shopping:latest -f docker/Dockerfile .'
                        sh 'docker tag shopping:latest kapilgurjar/shopping:latest'
                        sh 'docker push kapilgurjar/shopping:latest'
                    } 
                }
            }
        }
        
        stage('Docker Deploy to container') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '58df9d33-95db-4f0a-bd59-52954ae53708', toolName: 'docker') {
                        sh 'docker run -d --name shopping-cart -p 8070:8070 kapilgurjar/shopping:latest'
                    }
                }
            }
        }
    }
}

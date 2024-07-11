pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('GIT Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'a5987db9-b917-43c3-aa79-2b2892a36afc', poll: false, url: 'https://github.com/SubirKumar07/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
            sh "mvn clean compile -Dmaven.test.failure.ignore=true"
            }
        }
        stage('OWASP Scan') {
            steps {
            dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'OWASP'
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
       }
       stage('Sonarqube') {
            steps {
            withSonarQubeEnv('sonar-server'){
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=Shopping-Cart'''
               }
            }
       }
        stage('Build') {
            steps {
            sh "mvn clean package -Dmaven.test.failure.ignore=true"
            }
       }
        stage('Docker') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'd2adbb85-cc53-487f-a098-5204a6d37895', toolName: 'docker')
                    {
    
                    sh "docker build -t shopping-cart -f docker/Dockerfile ."
                    sh "docker tag shopping-cart subir15/shopping-cart:latest"
                    sh "docker push subir15/shopping-cart:latest"
                    }
                }
            
            }
       }
       
      stage('Deploy') {
            steps {
                 script{
                 withDockerRegistry(credentialsId: 'd2adbb85-cc53-487f-a098-5204a6d37895', toolName: 'docker')
                    {
                    sh "docker run -d --name shop-shop -p 8070:8070 subir15/shopping-cart:latest"
                    }
                 }
            }
       } 
   }
}

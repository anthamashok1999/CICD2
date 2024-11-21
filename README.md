pipeline {
    agent any
    
    tools {
        maven "maven3"
        jdk "jdk17"
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/anthamashok1999/Ekart.git'
            }
        }
        
           stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
           stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
            stage('SonarQube Analysis') {
            steps {
              withSonarQubeEnv('sonar') {
             sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART \
             -Dsonar.java.binaries=. '''
               }  
            }
        }
        
          stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
          stage('Deploy To Nexus') {
            steps {
             withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
              sh "mvn deploy -DskipTests=true"
               }   
            }
        }
    }
}

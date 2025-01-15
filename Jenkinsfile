pipeline{
    agent { label "jenkins agent"}
    tools {
        maven "maven3"
        jdk "java17"
    }
    stages {
        stage('cleanup workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Sourcecode checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Raviteja3399/register-app.git'
            }
        }
        stage('Build java code') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test the code') {
            steps {
                sh 'mvn test'
            } 
        }
        stage('Sonarqube Analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsid:'sonar creds'){
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage('quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar creds'
                }
            }
        }
    }
}

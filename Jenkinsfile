pipeline{
    agent { label "jenkins agent"}
    tools {
        maven "maven3"
        jdk "java17"
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "tejadocker333"
            DOCKER_PASS = 'dockerhub-creds'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
	stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image tejadocker333/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
    }
}

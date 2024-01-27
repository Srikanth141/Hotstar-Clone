pipeline {
    agent any
    tools{
        jdk 'JDK'
        nodejs 'NodeJS'
    }
    environment{
        SCANNER_HOME=tool 'SonarQube Scanner'
        APP_NAME = "hotstar"
        RELEASE = "1.0.0"
        DOCKER_USER = "srikanthmadhavarapu"
        DOCKER_PASS = "Docker"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Scm checkout') {
            steps {
                git branch: 'main', credentialsId: 'Github', url: 'https://github.com/Srikanth141/Hotstar-Clone.git'
            }
        }
        stage('sonar analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Hotstar \
                    -Dsonar.projectKey=Hotstar'''
                    
                }
            }
        }
        stage('sonarr-qualitygates') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy fs') {
            steps {
                sh 'trivy fs . > trivy.txt'
            }
        }
        stage('Docker build and push') {
            steps {
                script{
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image= docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        
                    }
                }
            }
        }
        stage('Trivy image scan') {
            steps {
                sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image srikanthmadhavarapu/hotstar:${IMAGE_TAG} --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest
                }
            }
        }
    }
}

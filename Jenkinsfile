pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'java17'
        maven 'maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "sohanmathewshera"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("jenkins-sonarqube-token")
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', 
                    credentialsId: 'github', 
                    url: 'https://github.com/sohanmathewshera/register-app'
            }
        }
      
        

        stage("Debug PATH") {
            steps {
                sh "echo $PATH"
                sh "which mvn || echo 'Maven not found in PATH'"
                sh "mvn -version || echo 'Maven not installed'"
            }
        }

        stage("Build Application") {
            steps {
                script {
                    def mvnHome = tool name: 'maven3', type: 'maven'
                    sh "${mvnHome}/bin/mvn clean package"
                }
            }
        }

      
        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
                    }
                }   
            }
        }
        
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }   
            }
        }
        
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build("${IMAGE_NAME}")
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh 'docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image sohanmathewshera/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table'
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

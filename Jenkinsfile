def frontendImage="olmy2016/frontend"
def backendImage="olmy2016/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def backendDockerTag=""
def frontendDockerTag=""

pipeline {
    agent { 
        label 'agent' 
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES=1
    }
    parameters {
        string 'backendDockerTag'
        string 'frontendDockerTag'
}
    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }
         stage('Adjust version') {
            steps {
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                   
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }

        }

        stage('Container rm') {
            steps {
                sh 'docker rm -f backend'
                sh 'docker rm -f frontend'
            }
        }
         stage('Deploy application') {
            steps {
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                             "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                       docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
                }
            }
         }

        stage('Tests') {
            steps {
                sh '''
                    pip3 install -r requirements.txt
                    python3 -m pytest ./test/selenium/*.py
                
                '''
            }

         
    } 
    post{
            always{
                sh 'docker-compose down'
                cleanWs()
            }
         }  
    }
}

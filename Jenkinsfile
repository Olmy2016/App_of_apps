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
    tools {
        terraform 'terraform'
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
                    pip3 install -r ./test/selenium/requirements.txt
                    python3 -m pytest ./test/selenium/frontendTest.py
                
                '''
            }
        }
        stage('Run terraform') {
            steps {
                dir('Terraform') {                
                    git branch: 'main', url: 'https://github.com/olmy2016/Terraform'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                            sh 'terraform init -backend-config=bucket=slawomir-skarba-panda-devops-core-17'
                            sh 'terraform apply -auto-approve -var bucket_name=slawomir-skarba-panda-devops-core-17'
                            
                    } 
                }
            }
        }
         
    } 
    post{
            always{
                sh 'docker-compose down'
                cleanWs()
            }
    }  
}

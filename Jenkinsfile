@Library('mySharedLibrary') _

def buildTag = ''

pipeline {
    agent { label 'ubuntu-agent' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master')
        booleanParam(name: 'DEPLOY', defaultValue: true)
    }

    environment {
        ENV = 'prod'  
        
    }

    stages {
        stage('Generate Tag') {
            steps {
                script {
                    buildTag = generateTag()
                }
            }
        }


        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/sriramkumar0604-lab/sampleApp', branch: "${params.BRANCH}"
            }
        }

         stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        export PATH=$PATH:/home/danish/.dotnet/tools

                        dotnet sonarscanner begin /k:"sampleapp"

                        dotnet build -c Release

                        dotnet sonarscanner end
                    '''
                }
            }
        }

        /*
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
      */
        
        stage('Build Docker Image') {
            steps {
                script {
                    buildDocker("$buildTag")
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    pushDocker("$buildTag")
                }
            }
        }

        stage('Deploy to Minikube') {
            when {
                expression { params.DEPLOY }
            }
            steps {
                script {

                    input message: "Do you want to deploy to Kubernetes?", ok: "Deploy"
                    echo "Deploying to environment: ${env.ENV}"
                    deployAKS("$buildTag")
                }
            }
        }
    }
}

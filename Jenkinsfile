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

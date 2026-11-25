pipeline {
    agent any

    environment {
        IMAGE_REPO = "the-sudip/cicd-pipeline"
        INTERNAL_PORT = "3000"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.HOST_PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.HOST_PORT = '3001'
                    } else {
                        env.HOST_PORT = '3002'
                    }
                    env.CONTAINER_NAME = "cicd-app-${env.BRANCH_NAME}"
                    env.IMAGE_TAG = "${env.IMAGE_REPO}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    echo "Branch: ${env.BRANCH_NAME} | Deploying on Host Port: ${env.HOST_PORT}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.IMAGE_TAG} ."
                sh "docker tag ${env.IMAGE_TAG} ${env.IMAGE_REPO}:${env.BRANCH_NAME}-latest"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker stop ${env.CONTAINER_NAME} || true
                    docker rm ${env.CONTAINER_NAME} || true
                    docker run -d --name ${env.CONTAINER_NAME} -p ${env.HOST_PORT}:${env.INTERNAL_PORT} ${env.IMAGE_TAG}
                """
            }
        }
    }
}
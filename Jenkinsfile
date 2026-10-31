pipeline {
    agent any

    tools {
        nodejs 'NodeJS-7.8.0'
    }

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test -- --watchAll=false'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.DOCKER_IMAGE = 'nodemain:v1.0'
                        env.CONTAINER_NAME = 'nodemain'
                        env.HOST_PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.DOCKER_IMAGE = 'nodedev:v1.0'
                        env.CONTAINER_NAME = 'nodedev'
                        env.HOST_PORT = '3001'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }

                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker rm -f ${CONTAINER_NAME} 2>/dev/null || true'
                sh 'docker run -d --restart unless-stopped --name ${CONTAINER_NAME} -p ${HOST_PORT}:3000 ${DOCKER_IMAGE}'
            }
        }
    }
}

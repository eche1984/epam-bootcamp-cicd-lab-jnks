pipeline {
    agent any
    
    tools {
        nodejs 'Node-7.8.0'
    }
    
    environment {
        DOCKER_IMAGE = ''
        HOST_PORT = ''
        CONTAINER_NAME = ''
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Set environment variables') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        DOCKER_IMAGE = 'nodemain:v1.0'
                        HOST_PORT = '3000'
                        CONTAINER_NAME = 'node-main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        DOCKER_IMAGE = 'nodedev:v1.0'
                        HOST_PORT = '3001'
                        CONTAINER_NAME = 'node-dev'
                    } else {
                        error "Branch ${env.BRANCH_NAME} not supported"
                    }
                    echo "Building for branch ${env.BRANCH_NAME}"
                    echo "Docker image: ${DOCKER_IMAGE}, Host port: ${HOST_PORT}"
                }
            }
        }
        
        stage('Build (npm install)') {
            steps {
                sh 'NODE_OPTIONS="--max-old-space-size=512" npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sh """
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    """
                    sh """
                        docker run -d --name ${CONTAINER_NAME} \
                        --expose ${HOST_PORT} -p ${HOST_PORT}:3000 \
                        ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}


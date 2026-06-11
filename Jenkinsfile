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
         
        stage('Push to Docker Hub') {
            steps {
                script {
                    def dockerHubRepo = "blackoctopus/epam-bootcamp-jnks-lab"
                    def imageTag = "${DOCKER_IMAGE}" 
                    def hubTag = imageTag.replace(':', '-')
                    def hubImage = "${dockerHubRepo}:${hubTag}"
                    sh "docker tag ${imageTag} ${hubImage}"
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${hubImage}"
                    }
                }
            }
        }
         
        stage('Trigger downstream') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main', wait: false
                    } else if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev', wait: false
                    }
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


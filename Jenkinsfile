@Library('cicd-shared-lib') _

pipeline {
    agent {
        docker {
            image 'blackoctopus/epam-bootcamp-jnks-lab:jenkins-agent-node'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
            reuseNode true
        }
    }
     
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
                        DOCKER_IMAGE = 'nodemain'
                        HOST_PORT = '3000'
                        CONTAINER_NAME = 'node-main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        DOCKER_IMAGE = 'nodedev'
                        HOST_PORT = '3001'
                        CONTAINER_NAME = 'node-dev'
                    } else {
                        error "Branch ${env.BRANCH_NAME} not supported"
                    }
                    IMAGE_TAG = 'v1.0'

                    echo "Building for branch ${env.BRANCH_NAME}"
                    echo "Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}, Host port: ${HOST_PORT}"
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
        
        stage('Hadolint Dockerfile Linter') {
            steps {
                script {
                    echo "🔍 Linting Dockerfile con Hadolint..."
                    sh """
                        hadolint \
                            --failure-threshold error \
                            --format tty \
                            Dockerfile
                    """
                    echo "✅ Dockerfile superó el linting sin errores"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                script {
                            // --exit-code 1 \
                    echo "Escaneando imagen ${DOCKER_IMAGE}:${IMAGE_TAG} con Trivy..."
                    sh """
                        trivy image \
                            --severity CRITICAL \
                            --exit-code 0 \
                            --no-progress \
                            ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                    echo "✅ No se encontraron vulnerabilidades CRITICAL o HIGH"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    dockerUtils(
                        image: DOCKER_IMAGE,
                        tag: IMAGE_TAG,
                        dockerUser: 'blackoctopus',
                        repoName: 'epam-bootcamp-jnks-lab'
                    )
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


pipeline {
    agent any
     environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    parameters {
        string(name: 'DOCKER_COMPOSE_FILE', defaultValue: 'docker-compose.yml', description: 'Path to the Docker Compose file')
    }

    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                git 'https://github.com/singhrajrohit/docker-frontend-backend-db.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh "docker-compose -f ${params.DOCKER_COMPOSE_FILE} build"
                }
            }
        }
        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=todo-app \
                   -Dsonar.sources=. \
                   -Dsonar.projectKey=todo-app '''
               }
            }
        }

        // Add more stages for testing, pushing, and deploying if needed
        stage('Push to Docker Hub') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('docker-cred') // ID of Docker Hub credentials in Jenkins
                DOCKERHUB_USERNAME = 'singhrajrohit' // Provide your Docker Hub username here
                REPOSITORY_NAME = 'mern_stack_application' // Provide the name of your target repository on Docker Hub
                IMAGE_TAG = 'latest' // Tag to be used for all images
            }
            steps {
                script {
                    // Log in to Docker Hub
                    sh "docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}"
                    
                    // Tag and push each service's image to Docker Hub
                    sh """
                        docker tag frontend-web:latest ${DOCKERHUB_USERNAME}/${REPOSITORY_NAME}:frontend-web-${IMAGE_TAG}
                        docker tag backend-api:latest ${DOCKERHUB_USERNAME}/${REPOSITORY_NAME}:backend-api-${IMAGE_TAG}
                        
                        
                        docker push ${DOCKERHUB_USERNAME}/${REPOSITORY_NAME}:frontend-web-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${REPOSITORY_NAME}:backend-api-${IMAGE_TAG}
                        
                    """
                }
            }
        }


        stage('Deploy') {
            steps {
                script {
                    // Stop and remove existing containers
                    sh "docker-compose -f ${params.DOCKER_COMPOSE_FILE} down"

                    // Start the containers
                    sh "docker-compose -f ${params.DOCKER_COMPOSE_FILE} up -d"
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline successful'
        }
        failure {
            echo 'CI/CD Pipeline failed'
        }
    }
}

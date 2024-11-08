pipeline {
    agent any

    environment {
        REACT_REPO = 'https://github.com/keeley-bootcamp/lbg-car-react-starter.git'
        SPRING_REPO = 'https://github.com/keeley-bootcamp/lbg-car-spring-app-starter.git'
        REACT_DIR = 'react'
        SPRING_DIR = 'spring'
        MAVEN_HOME = tool "M3"
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub_id'  // Jenkins credentials ID for Docker Hub
        DOCKERHUB_USERNAME = credentials('dockerhub_id') // Use credentials in Jenkins
        SSH_CREDENTIALS_ID = 'deploy-ssh-key'  // Jenkins SSH credentials ID for remote server
        REMOTE_SERVER = '35.210.185.254'  // The IP or hostname of the remote server
        REMOTE_PATH = '/home/gradc1_delegate9/group-3-pipeline'  // The directory on the remote server containing your docker-compose.yml
    }

    stages {
        // Stage 1: Checkout React repository
        stage('Checkout React') {
            steps {
                dir("${REACT_DIR}") {
                    git branch: 'main', url: "${REACT_REPO}"
                }
            }
        }

        // Stage 2: Install React dependencies
        stage('Install React Dependencies') {
            steps {
                dir("${REACT_DIR}") {
                    sh "npm install"
                }
            }
        }

        // Stage 3: Test React
        stage('Test React') {
            steps {
                dir("${REACT_DIR}") {
                    sh "npm test"
                }
            }
        }

        // Stage 4: Build React (production build)
        stage('Package React') {
            steps {
                dir("${REACT_DIR}") {
                    sh "npm run build"
                }
            }
        }

        // Stage 5: Checkout Spring Boot repository
        stage('Checkout Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    git branch: 'main', url: "${SPRING_REPO}"
                }
            }
        }

        // Stage 6: Compile Spring Boot app
        stage('Compile Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    sh "mvn clean compile"
                }
            }
        }

        // Stage 7: Test Spring Boot app
        stage('Test Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    sh "mvn test"
                }
            }
        }

        // Stage 8: Package Spring Boot app
        stage('Package Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    sh "mvn -Dmaven.test.skip -Dmaven.compile.skip package"
                }
            }
        }

        // Stage 9: Build Docker images for both React and Spring Boot apps
        stage('Build Docker Images') {
            parallel {
                stage('Build Docker Image for React') {
                    steps {
                        dir("${REACT_DIR}") {
                            script {
                                echo "Building Docker image for React..."
                                sh 'docker build -t keeleybootcamp/react-app .'
                            }
                        }
                    }
                }

                stage('Build Docker Image for Spring Boot') {
                    steps {
                        dir("${SPRING_DIR}") {
                            script {
                                echo "Building Docker image for Spring Boot..."
                                sh 'docker build -t keeleybootcamp/spring-app .'
                            }
                        }
                    }
                }
            }
        }

        // Stage 10: Push Docker images to Docker Hub
        stage('Push Docker Images to Docker Hub') {
            parallel {
                stage('Push React Image') {
                    steps {
                        script {
                            echo "Pushing React Docker image to Docker Hub..."
                            withDockerRegistry(credentialsId: DOCKERHUB_CREDENTIALS_ID) {
                                sh 'docker push keeleybootcamp/react-app'
                            }
                        }
                    }
                }

                stage('Push Spring Boot Image') {
                    steps {
                        script {
                            echo "Pushing Spring Boot Docker image to Docker Hub..."
                            withDockerRegistry(credentialsId: DOCKERHUB_CREDENTIALS_ID) {
                                sh 'docker push keeleybootcamp/spring-app'
                            }
                        }
                    }
                }
            }
        }

        // Stage 11: Deploy with Docker Compose on remote server
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    echo "Deploying applications using Docker Compose on remote server"

                    // SSH into the remote server and run docker-compose commands
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${REMOTE_SERVER} "
                                cd ${REMOTE_PATH} &&
                                docker-compose pull &&
                                docker-compose up -d
                            "
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Both React and Spring Boot applications have been successfully deployed to Docker Hub and containers are up and running on the remote server.'
        }
        failure {
            echo 'There was an error during the build or deployment process.'
        }
    }
}

pipeline {
    agent any

    environment {
        REACT_REPO = 'https://github.com/keeley-bootcamp/lbg-car-react-starter.git'
        SPRING_REPO = 'https://github.com/keeley-bootcamp/lbg-car-spring-app-starter.git'
        REACT_DIR = 'react'
        SPRING_DIR = 'spring'
        MAVEN_HOME = tool 'M3'
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub_id'  // Jenkins credentials ID for Docker Hub
        DOCKERHUB_USERNAME = credentials('dockerhub_id') // Use credentials in Jenkins
        SSH_CREDENTIALS_ID = 'deploy-ssh-key'  // Jenkins SSH credentials ID for remote server
        REMOTE_SERVER = '35.210.185.254'  // The IP or hostname of the remote server
        REMOTE_PATH = '/home/gradc1_delegate9/group-3-pipeline'  // The directory on the remote server containing your docker-compose.yml
        MYSQL_ROOT_PASSWORD = 'secretsecret'
        SERVER_URL = '35.210.185.254'
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

        stage('Test and build react frontend') {
            steps {
                dir("${REACT_DIR}") {
                    sh """
                    npm install
                    yarn test
                    docker build --build-arg SERVER_URL=${SERVER_URL} -t keeleybootcamp/react-app:v${BUILD_NUMBER} .
                    docker tag keeleybootcamp/react-app:v${BUILD_NUMBER} keeleybootcamp/react-app:latest
                    """
                }
            }
        }

        /*stage('Install React Dependencies') {
            steps {
                dir("${REACT_DIR}") {
                    sh "npm install"
                }
            }
        }

        stage('Test React') {
            steps {
                dir("${REACT_DIR}") {
                    sh "npm test"
                }
            }
        }

        stage('Package React') {
            steps {
                dir("${REACT_DIR}") {
                    sh "npm run build"
                }
            }
        }*/

        // Stage 5: Checkout Spring Boot repository
        stage('Checkout Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    git branch: 'main', url: "${SPRING_REPO}"
                }
            }
        }

        stage('Test and build spring backend') {
            steps {
                dir("lbg-car-back") {
                    sh "mvn clean test"
                    sh '''
                    cat - > src/main/resources/application.properties <<EOF
                    spring.profiles.active=prod
                    logging.level.root=DEBUG
                    server.port=8000
                    spring.jpa.show-sql=true
                    '''
                    sh "docker build -t keeleybootcamp/spring-app:v${BUILD_NUMBER} --build-arg MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} ."
                    sh "docker tag keeleybootcamp/spring-app:v${BUILD_NUMBER} keeleybootcamp/spring-app:latest"
                }
            }
        }

        /*stage('Compile Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    sh "mvn clean compile"
                }
            }
        }

        stage('Test Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    sh "mvn test"
                }
            }
        }

        stage('Package Spring') {
            steps {
                dir("${SPRING_DIR}") {
                    sh "mvn -Dmaven.test.skip -Dmaven.compile.skip package"
                }
            }
        }*/

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

}

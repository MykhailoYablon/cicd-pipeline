pipeline {
    agent docker

        environment {
            NODE_ENV = 'production'
            IMAGE_TAG = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
            PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
            APP_PORT = "3000"
        }

        tools {
            nodejs 'NodeJs Installation'
        }

        stages {

            stage('Checkout Code') {
                steps {
                    checkout scm
                }
            }

            stage('Tool Install') {
                steps {
                    sh 'npm install'
                }
            }

            stage('Run Tests') {
                steps {
                    sh 'npm test'
                }
            }

            stage('Docker build') {
                steps {
                    script {
                        sh "docker build -t ${IMAGE_TAG} ."
                    }
                }
            }

            stage('Deploy') {
                steps {
                    sh '''
                        echo "Looking for containers running on port ${PORT}..."
                        CONTAINER_ID=$(docker ps --filter "publish=${PORT}" --format "{{.ID}}")
                        if [ ! -z "$CONTAINER_ID" ]; then
                            echo "Stopping and removing container using port ${PORT}..."
                            docker stop $CONTAINER_ID || true
                            docker rm $CONTAINER_ID || true
                        else
                            echo "No container found using port ${PORT}"
                        fi
                    '''
                    sh '''
                        echo "Running app on port ${PORT}..."
                        docker run -d --expose ${PORT} -p ${PORT}:${APP_PORT} ${IMAGE_TAG}
                    '''
                }
            }
        }
}

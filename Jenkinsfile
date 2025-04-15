pipeline {
    agent any

        environment {
            NODE_ENV = 'production'
            IMAGE_TAG = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
            IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'ironcrow/nodemain:v1.0' : 'ironcrow/nodedev:v1.0'}"
            DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')
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

            stage('Lint Dockerfile with Hadolint') {
                steps {
                    sh 'hadolint Dockerfile'
                }
            }

            stage('Docker build') {
                steps {
                    script {
                        sh '''
                            echo "Pushing images to repo"
                            docker build -t ${IMAGE_TAG} .

                        '''
                    }
                }
            }

            stage('Scan Docker Image with Trivy') {
                steps {
                    script {
                        def imageName = (env.BRANCH_NAME == 'main') ? 'nodemain:v1.0' : 'nodedev:v1.0'
                        sh """
                        docker run --rm \
                          -v /var/run/docker.sock:/var/run/docker.sock \
                          aquasec/trivy:latest image \
                          --severity CRITICAL,HIGH ${imageName}
                        """
                    }
                }
            }

            stage('Push Image to Docker Hub') {
                steps {
                    script {
                        sh '''
                        echo "Pushing images to repo"
                        echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                        docker tag ${IMAGE_TAG} ${IMAGE_NAME}
                        docker push ${IMAGE_NAME}
                        '''
                    }
                }
            }

            stage('Trigger Deployment Pipeline') {
                steps {
                    script {
                        def targetPipeline = (env.BRANCH_NAME == 'main') ? 'Deploy_to_main' : 'Deploy_to_dev'
                        build job: targetPipeline, wait: false
                    }
                }
            }
        }
}
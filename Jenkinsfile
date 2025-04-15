pipeline {
    agent any

        environment {
            NODE_ENV = 'production'
            IMAGE_TAG = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
            IMAGE_NAME = 'ironcrow/devops-cicd'
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
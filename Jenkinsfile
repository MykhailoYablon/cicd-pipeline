pipeline {
    agent any

        environment {
            NODE_ENV = 'production'
            IMAGE_TAG = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
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

            stage('Push Image to Docker Hub') {
                steps {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        script {
                            IMAGE_NAME = (env.BRANCH_NAME == 'main') ? 'ironcrow/nodemain:v1.0' : 'ironcrow/nodedev:v1.0'

                            sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker tag ${IMAGE_TAG} ${IMAGE_NAME}
                            docker push ${IMAGE_NAME}
                            docker logout
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

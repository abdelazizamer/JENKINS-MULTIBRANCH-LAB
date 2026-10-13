pipeline { 
    agent any

    environment {
        APP_NAME = 'node-multibranch-app'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${APP_NAME}:${env.BRANCH_NAME}-${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        retry(3) {
                            sh """
                                echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                                docker tag ${APP_NAME}:${env.BRANCH_NAME}-${BUILD_NUMBER} ${DOCKER_USERNAME}/${APP_NAME}:${env.BRANCH_NAME}-${BUILD_NUMBER}
                                docker push ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                            """
                        }
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // تحديد بورت مخصص لكل فرع لمنع تضارب البورتات
                    def port = 5002
                    if (env.BRANCH_NAME == 'stg') {
                        port = 5003
                    } else if (env.BRANCH_NAME == 'pro') {
                        port = 5004
                    } else if (env.BRANCH_NAME == 'main') {
                        port = 5005
                    }

                    sh """
                        docker stop "${APP_NAME}-${env.BRANCH_NAME}-running" || true
                        docker rm "${APP_NAME}-${env.BRANCH_NAME}-running" || true
                        
                        docker run -d -p ${port}:5000 --name "${APP_NAME}-${env.BRANCH_NAME}-running" ${DOCKER_USERNAME}/${APP_NAME}:${env.BRANCH_NAME}-${BUILD_NUMBER}
                        docker ps
                    """
                }
            }
        }
    }
}
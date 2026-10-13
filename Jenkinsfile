pipeline { 
    agent any

    environment {
        APP_NAME = 'new-app'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${APP_NAME}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // استخدام retry لإعادة المحاولة تلقائياً لو حصل Timeout في النت
                        retry(3) {
                            sh """
                                echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                                docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
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
                    def port = 5000
                    if (env.BRANCH_NAME == 'stg') {
                        port = 5001
                    } else if (env.BRANCH_NAME == 'pro') {
                        port = 5002
                    }

                    sh """
                        docker stop "${APP_NAME}-${env.BRANCH_NAME}-running" || true
                        docker rm "${APP_NAME}-${env.BRANCH_NAME}-running" || true
                        docker run -d -p ${port}:5000 --name "${APP_NAME}-${env.BRANCH_NAME}-running" ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                        docker ps
                    """
                }
            }
        }
    }
}   
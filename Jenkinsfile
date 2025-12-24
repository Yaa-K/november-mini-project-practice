pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yaakesewaa/python-django-project"
        DOCKER_TAG   = "latest"
        EC2_HOST     = "ubuntu@51.20.85.35"
        APP_PORT     = "8000"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Login & Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKERHUB_USER',
                                                 passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh """
                      echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
                      docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKERHUB_USER',
                                                 passwordVariable: 'DOCKERHUB_PASS')]) {
                    sshagent(credentials: ['ec2-ssh-key']) {
                        sh """
                          ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                            echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop november-app || true
                            docker rm november-app || true
                            docker run -d -p ${APP_PORT}:${APP_PORT} --name november-app ${DOCKER_IMAGE}:${DOCKER_TAG}
                          '
                        """
                    }
                }
            }
        }
    }
}

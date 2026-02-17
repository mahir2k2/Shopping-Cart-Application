pipeline {
    agent { label 'agent-private-jnlp' }

    environment {
        DOCKER_IMAGE = "mahirs1205/aimdekassignment"
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_SERVER = "10.0.1.125"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    credentialsId: 'github-ssh-key',
                    url: 'git@github.com:mahir2k2/Shopping-Cart-Application.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                    docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['deploy-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} '
                            docker pull ${DOCKER_IMAGE}:${IMAGE_TAG} &&
                            docker stop shopping-app || true &&
                            docker rm shopping-app || true &&
                            docker run -d -p 3000:3000 \
                              --name shopping-app \
                              ${DOCKER_IMAGE}:${IMAGE_TAG}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}


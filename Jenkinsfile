pipeline {
    agent { label 'agent-private-jnlp' }

    environment {
        DOCKER_IMAGE = "mahirs1205/aimdekassignment"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/mehediislamripon/Shopping-Cart-Application.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
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
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['jenkins-agent-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@10.0.1.125 '
                            docker pull $DOCKER_IMAGE:$IMAGE_TAG &&
                            docker stop shopping-app || true &&
                            docker rm shopping-app || true &&
                            docker run -d -p 3000:3000 --name shopping-app $DOCKER_IMAGE:$IMAGE_TAG
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


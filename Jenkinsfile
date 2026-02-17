pipeline {
    agent { label 'agent-private-jnlp' }

    environment {
        DOCKER_IMAGE = "mahirs1205/aimdekassignment"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_REGION = "ap-south-1"
        INSTANCE_ID = "i-0e8bd4b5e9e8ca9d1"
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

        stage('Deploy to EC2 via SSM') {
            steps {
                sh """
                    aws ssm send-command \
                      --instance-ids ${INSTANCE_ID} \
                      --document-name "AWS-RunShellScript" \
                      --comment "Deploying Docker container via Jenkins" \
                      --parameters commands="
                        docker pull ${DOCKER_IMAGE}:${IMAGE_TAG};
                        docker stop shopping-app || true;
                        docker rm shopping-app || true;
                        docker run -d -p 3000:3000 --name shopping-app ${DOCKER_IMAGE}:${IMAGE_TAG};
                      " \
                      --region ${AWS_REGION}
                """
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


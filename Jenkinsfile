pipeline {
    agent any

    tools {
        nodejs 'Node18'
    }

    environment {
        WEB_IP = '13.219.91.69'
        SSH_CRED = 'deploy-ec2-key'
        DOCKER_IMAGE = 'abhishek0225/my-react-app:latest'
        CONTAINER_NAME = 'reactapp'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install dependencies') {
            steps { sh 'npm ci || npm install' }
        }

        stage('Build React App') {
            steps { sh 'npm run build' }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${DOCKER_IMAGE} .
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${WEB_IP} '
                      docker pull ${DOCKER_IMAGE};
                      docker stop ${CONTAINER_NAME} || true;
                      docker rm ${CONTAINER_NAME} || true;
                      docker run -d --name ${CONTAINER_NAME} -p 80:80 ${DOCKER_IMAGE};
                    '
                    """
                }
            }
        }
    }

    post {
        success { echo '✅ Deployment completed successfully!' }
        failure { echo '❌ Deployment failed. Check the logs.' }
    }
}

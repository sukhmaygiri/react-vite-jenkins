pipeline {

    agent any

    environment {
        IMAGE_NAME = "sukhmay/react-vite-jenkins"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = "/var/lib/jenkins/kubeconfig"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    )
                ]) {
                    sh '''
                    echo $PASSWORD | docker login -u $USERNAME --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "===== Current User ====="
                whoami

                echo "===== AWS Identity ====="
                aws sts get-caller-identity

                echo "===== Kubernetes Nodes ====="
                kubectl get nodes

                echo "===== Updating Deployment Image ====="
                kubectl set image deployment/react-vite-deployment react-vite=$IMAGE_NAME:$IMAGE_TAG

                echo "===== Waiting for Rollout ====="
                kubectl rollout status deployment/react-vite-deployment

                echo "===== Services ====="
                kubectl get svc

                echo "===== Pods Status ====="
                kubectl get pods
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful'
        }

        failure {
            echo '❌ Deployment Failed'
        }
    }
}
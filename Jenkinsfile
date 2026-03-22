pipeline {
    agent any

    environment {
        IMAGE_NAME = "node-app"
        DOCKERHUB_USER = "prabha0112"
        CONTAINER_NAME = "node-app-container"
        KUBE_DEPLOYMENT = "node-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image..."
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "Logging into Docker Hub..."
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                echo "Pushing image..."
                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "Deploying to Kubernetes..."

                kubectl apply -f k8s/

                # Restart deployment to pull latest image
                kubectl rollout restart deployment/$KUBE_DEPLOYMENT
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Checking pods..."
                kubectl get pods

                echo "Checking services..."
                kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment to Kubernetes Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}

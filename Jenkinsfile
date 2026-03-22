pipeline {
    agent any

    environment {
        IMAGE_NAME = "node-app"
        DOCKERHUB_USER = "prabha0112"   // your Docker Hub username
        KUBE_DEPLOYMENT = "node-app"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
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
                echo "Pushing Docker image..."
                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "Deploying to Kubernetes..."

                export KUBECONFIG=$KUBECONFIG

                kubectl get nodes

                kubectl apply -f k8s/

                kubectl rollout restart deployment/$KUBE_DEPLOYMENT
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                export KUBECONFIG=$KUBECONFIG

                echo "Pods status:"
                kubectl get pods

                echo "Service status:"
                kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}

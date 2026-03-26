pipeline {
    agent any

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yashodaadarsh/DevOpsLab-CICD'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t my-k8s-node-app:${BUILD_NUMBER} .

                docker tag my-k8s-node-app:${BUILD_NUMBER} yashodaadarsh/my-k8s-node-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker push yashodaadarsh/my-k8s-node-app:${BUILD_NUMBER}

                    docker logout
                    '''
                }
            }
        }


        // OPTIONAL: Kubernetes stages (enable when needed)

        
        stage('Start Minikube if not running') {
            steps {
                sh '''
                if ! minikube status | grep -q "apiserver: Running"; then
                    echo "Starting Minikube..."
                    minikube start --driver=docker
                fi
                '''
                //  minikube start --driver=docker --memory=2048 --cpus=2
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Load image into Minikube
                minikube image load yashodaadarsh/my-k8s-node-app:${BUILD_NUMBER}

                # Update deployment with new image
                sed -i "s|image:.*|image: yashodaadarsh/my-k8s-node-app:${BUILD_NUMBER}|g" k8s/deployment.yaml

                # Apply manifests
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify deployment'){
            steps{
                sh '''
                    minikube service my-k8s-app-service --url
                '''
            }
        }
        

    }
}
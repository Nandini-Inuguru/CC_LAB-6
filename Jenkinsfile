pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Removing old backend image if exists..."
                docker rmi -f backend-app || true

                echo "Building backend Docker image..."
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Creating Docker network..."
                docker network create app-network || true

                echo "Removing old backend containers..."
                docker rm -f backend1 backend2 || true

                echo "Starting backend containers..."
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Removing old nginx container..."
                docker rm -f nginx-lb || true

                echo "Starting nginx load balancer..."
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf \
                  nginx
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
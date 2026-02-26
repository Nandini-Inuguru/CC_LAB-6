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
                echo "Creating Docker network if not exists..."
                docker network inspect app-network >/dev/null 2>&1 || docker network create app-network

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
                  nginx

                echo "Copying nginx configuration..."
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                echo "Restarting nginx container to apply config..."
                docker restart nginx-lb
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
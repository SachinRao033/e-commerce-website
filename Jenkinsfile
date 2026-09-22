pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/e-commerce-website"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Project') {
            steps {
                sh '''
                sudo mkdir -p "$PROJECT_DIR"
                sudo chmod 755 /home/ubuntu

                sudo rsync -av --delete \
                    --exclude='.git' \
                    "$WORKSPACE"/ "$PROJECT_DIR"/

                sudo chown -R jenkins:jenkins "$PROJECT_DIR"

                echo "Project copied successfully"
                ls -la "$PROJECT_DIR"
                '''
            }
        }

        stage('Create Environment Files') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                cat > .env <<EOF
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123
POSTGRES_DB=ecommerce
REACT_APP_BACKEND_URL=http://16.4.19.228:8000
EOF

                mkdir -p backend
                cat > backend/.env.docker <<EOF
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123
POSTGRES_DB=ecommerce
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
EOF

                mkdir -p frontend
                cat > frontend/.env.docker <<EOF
REACT_APP_BACKEND_URL=http://16.4.19.228:8000
EOF

                echo "Environment files created"
                '''
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose down || true
                docker system prune -af --volumes || true
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose build --no-cache
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Waiting for services..."
                sleep 20

                cd "$PROJECT_DIR"

                echo "===== Docker Compose Status ====="
                docker compose ps

                echo "===== Running Containers ====="
                docker ps

                echo "===== Backend Health ====="
                curl -f http://localhost:8000/docs > /dev/null

                echo "Backend is healthy"
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS: E-Commerce Website deployed successfully!"
        }

        failure {
            echo "FAILED: Deployment failed. Check Jenkins console output."
        }

        always {
            sh '''
            sudo chown -R ubuntu:ubuntu "$PROJECT_DIR" || true
            docker image prune -f || true
            '''
        }
    }
}

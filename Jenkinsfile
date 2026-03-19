pipeline {
    agent any

    environment {
        // Your Docker Hub details
        DOCKER_HUB_USERNAME = 'leigha12345'
        IMAGE_NAME = "${DOCKER_HUB_USERNAME}/runcalc-pro"
        
        // Your GitHub details
        REPO_URL = 'https://github.com/Leigha1234/devops-lab-9.git'
        REPO_BRANCH = 'main'
        
        // Folder containing the FastAPI code
        APP_DIR = 'flask-app'
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls the latest code from your GitHub
                git branch: "${REPO_BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Test') {
            steps {
                sh """
                    cd ${APP_DIR}
                    python3 -m venv ./venv
                    . ./venv/bin/activate
                    pip install -r requirements.txt
                    # Note: You can add 'pytest' here if you have tests written
                """
            }
        }

        stage('Build') {
            steps {
                sh """
                    cd ${APP_DIR}
                    # Build the image using your Dockerfile
                    docker build -t ${IMAGE_NAME}:latest .
                    # Tag it with the specific Jenkins build number
                    docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:v1.${env.BUILD_NUMBER}
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // This uses the 'docker-hub-token' you must set up in Jenkins Credentials
                withCredentials([string(
                    credentialsId: 'docker-hub-token',
                    variable: 'DOCKER_HUB_TOKEN'
                )]) {
                    sh """
                        echo "$DOCKER_HUB_TOKEN" | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin
                        docker push ${IMAGE_NAME}:v1.${env.BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up the local images to save disk space on your Jenkins server
            sh "docker rmi ${IMAGE_NAME}:v1.${env.BUILD_NUMBER} ${IMAGE_NAME}:latest || true"
        }
    }
}
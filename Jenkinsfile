pipeline {
    agent any

    enviroment {
        DOCKERHUB_USER = "khanhdang21"
        APP_NAME = "task-manager-mini-fastapi"
        IMAGE_NAME     = "${DOCKERHUB_USER}/${APP_NAME}"
        IMAGE_TAG      = "${env.BUILD_NUMBER}" 
    }

    stages {
        stage('Checkout') {
            steps {
                chechout scm
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        docket logout
                    """
                }
            }
        }

        stage('Pull and Deploy') {
            steps {
                withCredentials([file(
                    credentialsId: 'task-manager-env-file', 
                    variable: 'ENV_FILE'
                )]) {
                    sh """
                        cp \$ENV_FILE .env
                        docker pull ${IMAGE_NAME}:latest
                        docker compose down
                        docker compose up -d
                        rm -f .env
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deploy successfully: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Failed"
        }
        always {
            sh "docker image prune -f || true"
            sh "docker logout || true"
            sh "rm -f .env || true"   
        }
    }
}
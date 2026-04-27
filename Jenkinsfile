pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDS = 'dockerhub-creds'
        GCP_CREDS = 'gcp-credentials' 
        GCP_PROJECT = 'fastapi-terraform-lab'
        GCP_CLUSTER = 'my-cluster' 
        GCP_REGION = 'us-central1'
        DOCKER_IMAGE = 'larelein/notejam' 
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Додаємо 'def', щоб Jenkins не сварився на нову змінну
                    // Використовуємо глобальну змінну env, щоб передати її далі
                    env.DOCKER_TAG = "${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${env.DOCKER_TAG} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_HUB_CREDS) {
                        // Використовуємо прямі команди тегування та пушу
                        sh "docker tag ${env.DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                        sh "docker push ${env.DOCKER_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    // Використовуємо одинарні лапки для sh, щоб Groovy не чіпав знаки долара
                    sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                    sh 'chmod +x ./kubectl'
                    
                    // Тут можна залишити подвійні лапки, бо ми використовуємо змінні Jenkins
                    sh "gcloud container clusters get-credentials ${GCP_CLUSTER} --region ${GCP_REGION} --project ${GCP_PROJECT}"
                    
                    // Деплой
                    sh './kubectl apply -f kubernetes.yaml'
                    
                    // Перевірка
                    sh './kubectl get pods'
                    sh './kubectl get svc web'
                }
            }
        }
    }
}

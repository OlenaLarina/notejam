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
                    // 1. Встановлюємо kubectl через gcloud (це надійніше в GCE)
                    sh "gcloud components install kubectl --quiet"
                    
                    // 2. Отримуємо доступ до кластера
                    sh "gcloud container clusters get-credentials ${GCP_CLUSTER} --region ${GCP_REGION} --project ${GCP_PROJECT}"
                    
                    // 3. Застосовуємо конфігурацію (вказуємо повний шлях до kubectl)
                    // Ми додаємо шлях до компонентів gcloud, щоб система бачила kubectl
                    sh "export PATH=$PATH:/usr/lib/google-cloud-sdk/bin && kubectl apply -f kubernetes.yaml"
                    
                    // 4. Перевірка
                    sh "export PATH=$PATH:/usr/lib/google-cloud-sdk/bin && kubectl get pods"
                }
            }
        }
    }
}

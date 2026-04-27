pipeline {
    agent any

    environment {
        PROJECT_ID = 'fastapi-terraform-lab' // Ваш ID проекту
        CLUSTER_NAME = 'my-cluster'         // Ваша назва кластера
        LOCATION = 'us-central1'            // Ваш регіон
        DOCKER_IMAGE = 'larelein/notejam'   // Ваш образ
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push') {
            steps {
                script {
                    // Збираємо образ
                    def app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                    
                    // Пушимо в Docker Hub (використовуємо ваш існуючий запис 'dockerhub-creds')
                    docker.withRegistry('', 'dockerhub-creds') {
                        app.push()
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    // 1. Завантажуємо kubectl (працює)
                    sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                    sh 'chmod +x ./kubectl'

                    // 2. Оновлюємо тег (працює)
                    sh "sed -i 's|larelein/notejam:latest|larelein/notejam:${env.BUILD_NUMBER}|g' kubernetes.yaml"

                    // --- ТРЮК ТУТ ---
                    // Примусово кажемо gcloud використовувати акаунт машини для авторизації
                    sh "gcloud auth login --brief --activate --quiet || true" 
                    sh "gcloud config set project fastapi-terraform-lab"
                    // ----------------

                    // 3. Отримання доступів
                    sh "gcloud container clusters get-credentials my-cluster --region us-central1 --project fastapi-terraform-lab"

                    // 4. Деплой через локальний kubectl
                    sh "./kubectl apply -f kubernetes.yaml"
                    sh "./kubectl get pods"
                }
            }
        }
    }
}

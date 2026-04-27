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
            
            // 2. Підготовка маніфесту
            sh "sed -i 's|larelein/notejam:latest|larelein/notejam:${env.BUILD_NUMBER}|g' kubernetes.yaml"

            // 3. СИЛОВЕ ОНОВЛЕННЯ АВТОРИЗАЦІЇ
            // Видаляємо стару конфігурацію, щоб gcloud отримав новий токен
            sh "rm -rf ~/.config/gcloud"
            sh "gcloud config set project fastapi-terraform-lab"
            sh "gcloud config set container/use_client_certificate False"
            
            // Дивимось, під ким ми зараз (для діагностики)
            sh "gcloud auth list"

            // 4. Отримання доступів
            sh "gcloud container clusters get-credentials my-cluster --region us-central1 --project fastapi-terraform-lab"

            // 5. Деплой
            sh "./kubectl apply -f kubernetes.yaml"
            sh "./kubectl get pods"
        }
    }
}
    }
}

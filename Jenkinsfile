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
            // 1. Завантаження kubectl (це у вас вже працює добре)
            sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
            sh 'chmod +x ./kubectl'
            
            // 2. Оновлення тегу образу
            sh "sed -i 's|larelein/notejam:latest|larelein/notejam:${env.BUILD_NUMBER}|g' kubernetes.yaml"

            // 3. ОТРИМАННЯ ТОКЕНА
            // Ми беремо токен, який Google видає вашій машині
            def token = sh(script: 'gcloud auth print-access-token', returnStdout: true).trim()

            // 4. РУЧНЕ НАЛАШТУВАННЯ КУБЕРНЕТЕС (Вставляємо ваш IP сюди)
            // Ми кажемо kubectl: "Ось адреса сервера, не перевіряй сертифікати, просто підключись"
            sh "./kubectl config set-cluster my-cluster --server=https://34.136.118.236 --insecure-skip-tls-verify=true"
            
            // Підставляємо токен для авторизації
            sh "./kubectl config set-credentials jenkins --token=${token}"
            
            // Створюємо контекст для підключення
            sh "./kubectl config set-context default --cluster=my-cluster --user=jenkins"
            sh "./kubectl config use-context default"

            // 5. ФІНАЛЬНИЙ ДЕПЛОЙ
            // Тепер kubectl не буде питати gcloud про доступи, а використає те, що ми дали
            sh "./kubectl apply -f kubernetes.yaml"
            sh "./kubectl get pods"
        }
    }
}
    }
}

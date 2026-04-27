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
                    // 1. Завантажуємо kubectl (це ми вже перевірили, працює)
                    sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                    sh 'chmod +x ./kubectl'

                    // 2. Оновлюємо тег образу у файлі маніфесту (як у методичці через sed)
                    // Замінюємо заглушку в kubernetes.yaml на реальний номер білду
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' kubernetes.yaml"

                    // 3. Авторизація через права самої VM (Full Access)
                    sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${LOCATION} --project ${PROJECT_ID}"

                    // 4. Деплой
                    sh "./kubectl apply -f kubernetes.yaml"
                    
                    echo "Deployment finished successfully!"
                }
            }
        }
    }
}

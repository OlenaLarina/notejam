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
            sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
            sh 'chmod +x ./kubectl'
            sh "sed -i 's|larelein/notejam:latest|larelein/notejam:${env.BUILD_NUMBER}|g' kubernetes.yaml"

            // 1. Отримуємо токен і IP кластера
            def token = sh(script: 'gcloud auth print-access-token', returnStdout: true).trim()
            // ДІЗНАЙТЕСЯ IP ВАШОГО КЛАСТЕРА (Endpoint) в консолі GCP і вставте сюди замість CLUSTER_IP
            def clusterIp = sh(script: "gcloud container clusters describe my-cluster --region us-central1 --format='get(endpoint)'", returnStdout: true).trim()

            // 2. Налаштовуємо kubectl напряму, минаючи get-credentials
            sh "./kubectl config set-cluster my-cluster --server=https://${clusterIp} --insecure-skip-tls-verify=true"
            sh "./kubectl config set-credentials jenkins --token=${token}"
            sh "./kubectl config set-context default --cluster=my-cluster --user=jenkins"
            sh "./kubectl config use-context default"

            // 3. ФІНАЛЬНИЙ ДЕПЛОЙ
            sh "./kubectl apply -f kubernetes.yaml"
            sh "./kubectl get pods"
        }
    }
}
    }
}

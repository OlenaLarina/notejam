pipeline {
    agent any

    environment {
        // Тут ми вказуємо назви ключів, які ви щойно створили в Jenkins
        DOCKER_HUB_CREDS = 'dockerhub-creds'
        GCP_CREDS = 'gcp-credentials' // це ваш Metadata-ключ
        GCP_PROJECT = 'fastapi-terraform-lab'
        GCP_CLUSTER = 'my-cluster' // перевірте назву вашого кластера в Google Console
        GCP_REGION = 'us-central1'
        DOCKER_IMAGE = 'larelein/notejam' // змініть 'olena-larina' на ваш логін Docker Hub
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Завантаження коду з вашого GitHub
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Збірка образу на основі вашого Dockerfile
                    app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Авторизація та відправка образу в репозиторій
                    docker.withRegistry('', DOCKER_HUB_CREDS) {
                        app.push()
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                // Деплой у ваш кластер Kubernetes
                step([
                    $class: 'GKEPublisher',
                    credentialsId: env.GCP_CREDS,
                    project: env.GCP_PROJECT,
                    clusterName: env.GCP_CLUSTER,
                    region: env.GCP_REGION,
                    manifestPattern: 'kubernetes.yaml' // цей файл теж має бути в репозиторії
                ])
            }
        }
    }
}

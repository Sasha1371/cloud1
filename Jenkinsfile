pipeline {
    agent none  // Загальний агент для всього pipeline

    environment {
        KUBE_CREDENTIALS_ID = '1625499a-1f81-4740-a265-6d1d81993962'
        KUBE_SERVER_URL = 'https://420447844CB1572AF2E75754D904D9A1.gr7.eu-central-1.eks.amazonaws.com'
    }

    stages {
        stage('Підключення до Kubernetes') {
            steps {
                kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL) {
                    echo 'Kubernetes connection established'
                }
            }
        }

        stage('Запуск MySQL на Node 1') {
            agent { label 'k8s-node-1' }
            steps {
                script {
                    docker.image('mcr.microsoft.com/mssql/server:2022-latest').withRun("-e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=Qwerty-1 -p 1433:1433 --name sql111 --hostname sql1") {
                        // Додаткові налаштування
                    }
                }
            }
        }

        stage('Запуск FrontEnd на Node 2') {
            agent { label 'k8s-node-2' }
            steps {
                kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL) {
                    script {
                        docker.build("FrontEnd/my-app/", "-t frontend:latest")
                        docker.container().run("-d -p 81:80 frontend:latest")
                    }
                }
            }
        }

        stage('Запуск BackEnd на Node 3') {
            agent { label 'k8s-node-3' }
            steps {
                kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL) {
                    script {
                        docker.build("BackEnd/Amazon-clone/", "-t backend:latest")
                        docker.container().run("-d -p 5034:5034 backend:latest")
                    }
                }
            }
        }
    }
}

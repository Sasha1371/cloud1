pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent-cluster'
            defaultContainer 'jnlp'
        }
    environment {
        KUBE_CREDENTIALS_ID = '1625499a-1f81-4740-a265-6d1d81993962'
        KUBE_SERVER_URL = 'https://420447844CB1572AF2E75754D904D9A1.gr7.eu-central-1.eks.amazonaws.com'
    }

    stages {
        stage('Запуск MySQL на кластері') {
            steps {
                container('docker') {  // Використовуємо контейнер з Docker
                    script {
                        sh 'docker run -d -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name sql111 --hostname sql1 mcr.microsoft.com/mssql/server:2022-latest'
                        echo 'MySQL container started'
                    }
                }
            }
        }

        stage('Build FrontEnd') {
            steps {
                dir('FrontEnd/my-app') {
                    container('docker') {  // Використовуємо контейнер з Docker
                        script {
                            sh 'docker build -t frontend:latest .'
                            sh 'docker run -d -p 81:80 frontend:latest'
                        }
                    }
                }
            }
        }

        stage('Build BackEnd') {
            steps {
                dir('BackEnd/Amazon-clone') {
                    container('docker') {  // Використовуємо контейнер з Docker
                        script {
                            sh 'docker build -t backend:latest .'
                            sh 'docker run -d -p 5034:5034 backend:latest'
                        }
                    }
                }
            }
        }
    }
}

pipeline {
    agent {
        kubernetes {
            inheritFrom 'cluster-agent'  // Назва Pod Template, який вже існує в Jenkins
            defaultContainer 'jnlp'
        }
    }

    environment {
        KUBE_CREDENTIALS_ID = '1625499a-1f81-4740-a265-6d1d81993962'
        KUBE_SERVER_URL = 'https://420447844CB1572AF2E75754D904D9A1.gr7.eu-central-1.eks.amazonaws.com'
    }

    stages {
        stage('Запуск MySQL на кластері') {
            steps {
                script {
                    def mysqlImage = docker.image('mcr.microsoft.com/mssql/server:2022-latest')
                    mysqlImage.run('-e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name sql111 --hostname sql1 -d')
                    echo 'MySQL container started'
                }
            }
        }

        stage('Build FrontEnd') {
            steps {
                dir('FrontEnd/my-app') {
                    script {
                        def frontendImage = docker.build("frontend/my-app", "-t frontend:latest .")
                        frontendImage.run('-d -p 81:80')
                    }
                }
            }
        }

        stage('Build BackEnd') {
            steps {
                dir('BackEnd/Amazon-clone') {
                    script {
                        def backendImage = docker.build("backend/my-app", "-t backend:latest .")
                        backendImage.run('-d -p 5034:5034')
                    }
                }
            }
        }
    }
}

pipeline {
    agent none

    environment {
        KUBE_CREDENTIALS_ID = '1625499a-1f81-4740-a265-6d1d81993962'
        KUBE_SERVER_URL = 'https://420447844CB1572AF2E75754D904D9A1.gr7.eu-central-1.eks.amazonaws.com'
        KUBE_CA_CERTIFICATE = '''<YOUR_CERTIFICATE>'''
    }

    stages {
        
        stage('Підключення до Kubernetes') {
            agent { label 'k8s-node-1' }
            steps {
                kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL, caCertificate: KUBE_CA_CERTIFICATE) {
                    echo 'Kubernetes connection established'
                }
            }
        }

        stage('Запуск MySQL на Node 1') {
            agent { label 'k8s-node-1' }
            steps {
                script {
                    docker.image('mcr.microsoft.com/mssql/server:2022-latest').run("-d -e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=Qwerty-1 -p 1433:1433 --name sql111 --hostname sql1")
                    echo 'MySQL container started'
                }
            }
        }

        stage('Запуск FrontEnd на Node 2') {
            agent { label 'k8s-node-2' }
            steps {
                script {
                    kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL, caCertificate: KUBE_CA_CERTIFICATE) {
                        dir('FrontEnd/my-app') {
                            docker.build(".", "-t frontend:latest")
                            docker.image("frontend:latest").run("-d -p 81:80")
                            echo 'FrontEnd container started'
                        }
                    }
                }
            }
        }

        stage('Запуск BackEnd на Node 3') {
            agent { label 'k8s-node-3' }
            steps {
                script {
                    kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL, caCertificate: KUBE_CA_CERTIFICATE) {
                        dir('BackEnd/Amazon-clone') {
                            docker.build(".", "-t backend:latest")
                            docker.image("backend:latest").run("-d -p 5034:5034")
                            echo 'BackEnd container started'
                        }
                    }
                }
            }
        }
    }
}

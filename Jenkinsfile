pipeline {
    agent none  // Загальний агент для всього pipeline

    environment {
        KUBE_CREDENTIALS_ID = '1625499a-1f81-4740-a265-6d1d81993962'
        KUBE_SERVER_URL = 'https://420447844CB1572AF2E75754D904D9A1.gr7.eu-central-1.eks.amazonaws.com'
        KUBE_CA_CERTIFICATE = '''
-----BEGIN CERTIFICATE-----
MIIDBTCCAe2gAwIBAgIIesQYdtIazdYwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yNDA5MjcyMzQ2MjRaFw0zNDA5MjUyMzUxMjRaMBUx
EzARBgNVBAMTCmt1YmVybmV0ZXMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCtlwwXDtpVqkECeHOHZIe3rIOtRQIUI9GAZfTizdVArECdt6FacsKZqPcT
HQ8c9uguymEhkfHZabbJX6Im3R9SQpiutgRUr8SDMcGQBZHJEpZDK/pwEbX+nu9F
M6wxpvWfW0cwFwBqg4+BaH3OkBeH41EFn8G+NQsJMDKf38SY/0GL0dtcUH/Z7ZBY
9yLpB/xsUgcDuXgXWHLMsbOodqRl2M0mKBLrmTcE/deJHg5yk5OnurvYkYmRUSx6
f7Tc1cWX0bEDLRs+pW+2C+a37IEsqqnyhtQLWEJ6kczJdjDzmZm5j6nR1ldxMjAo
k3egmykV2utgsUdhaRVYiuhKAz9NAgMBAAGjWTBXMA4GA1UdDwEB/wQEAwICpDAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBT0FCZ3bs9r3x7jxu/iurOGg1dekjAV
BgNVHREEDjAMggprdWJlcm5ldGVzMA0GCSqGSIb3DQEBCwUAA4IBAQB4NCUhVO/a
w3NOTmGNlbAl9JA70ZHkkWJh6ZcnTX2aczAe1OLg7uDagyZffYV9G5aOWm1FCSid
tI+hJHFCFr43W3dZdcbAj+4m0vZiqSCM1PxHjAHWB7hms/5MnG7AdGVJXkS+XRUw
DM/CUV3Ok0kYqrmpezH6k/sHltJbz7oGfOH1aRgWMJQiGH/4lT8s1dGgJNkmTeDa
YZU0qzF8q2gWSxh3N6OIC5gu2eReLEHdzTfZOGuy2/x2sNkLGJis363V16+fGGcg
ZSJrHarDVQvoibbDDLllyS26Fm/7fWmFZwZCaat19jJ31xRZvv1fZ9qX/G2cPjfS
kw10izU57mb9
-----END CERTIFICATE-----
'''
    }

    stages {
        stage('Підключення до Kubernetes') {
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
                    docker.image('mcr.microsoft.com/mssql/server:2022-latest').withRun("-e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=Qwerty-1 -p 1433:1433 --name sql111 --hostname sql1") {
                        // Додаткові налаштування
                    }
                }
            }
        }

        stage('Запуск FrontEnd на Node 2') {
            agent { label 'k8s-node-2' }
            steps {
                kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL, caCertificate: KUBE_CA_CERTIFICATE) {
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
                kubeconfig(credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL, caCertificate: KUBE_CA_CERTIFICATE) {
                    script {
                        docker.build("BackEnd/Amazon-clone/", "-t backend:latest")
                        docker.container().run("-d -p 5034:5034 backend:latest")
                    }
                }
            }
        }
    }
}

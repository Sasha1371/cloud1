pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'bf804a0d-eb49-4d69-a3b3-f0950605b494'
        SQL_CONTAINER_NAME = 'lendy123/sql'
        FRONTEND_CONTAINER_NAME = 'lendy123/frontend'
        BACKEND_CONTAINER_NAME = 'lendy123/backend'
    }

    stages {
        stage('Вход в DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        sh "echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Билд и запуск MySQL контейнера') {
            steps {
                script {
                    docker.image('mcr.microsoft.com/mssql/server:2022-latest').withRun("-e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=Qwerty-1 -p 1433:1433 --name ${SQL_CONTAINER_NAME} --hostname sql1") {
                        // Optional: Perform any additional setup after the container starts
                    }
                }
            }
        }

        stage('Тегирование и пуш MySQL образа в DockerHub') {
            steps {
                script {
                    docker.image('mcr.microsoft.com/mssql/server:2022-latest').tag("lendy123/sql:latest").push()
                }
            }
        }

        stage('Билд и пуш FrontEnd образа в DockerHub') {
            steps {
                script {
                    docker.build("FrontEnd/my-app/", "-t lendy123/frontend:version${BUILD_NUMBER}").push()
                    docker.image("lendy123/frontend:version${BUILD_NUMBER}").tag("lendy123/frontend:latest").push()
                }
            }
        }

        stage('Билд и пуш BackEnd образа в DockerHub') {
            steps {
                script {
                    docker.build("BackEnd/Amazon-clone/", "-t lendy123/backend:version${BUILD_NUMBER}").push()
                    docker.image("lendy123/backend:version${BUILD_NUMBER}").tag("lendy123/backend:latest").push()
                }
            }
        }

        stage('Остановка и удаление старых контейнеров') {
            steps {
                script {
                    docker.container(FRONTEND_CONTAINER_NAME).stop().remove(force: true)
                    docker.container(BACKEND_CONTAINER_NAME).stop().remove(force: true)
                    docker.container(SQL_CONTAINER_NAME).stop().remove(force: true)
                }
            }
        }

        stage('Чистка старых образов') {
            steps {
                script {
                    docker.image().prune(all: true, filter: "until=24h")
                }
            }
        }

        stage('Запуск Docker контейнеров') {
            steps {
                script {
                    docker.container().run("-d -p 81:80 lendy123/frontend")
                    docker.container().run("-d -p 5034:5034 lendy123/backend")
                }
            }
        }
    }
}

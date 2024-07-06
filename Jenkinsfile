pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'bf804a0d-eb49-4d69-a3b3-f0950605b494'
        SQL_CONTAINER_NAME = 'lendy123/sql'
        FRONTEND_CONTAINER_NAME = 'lendy123/frontend'
        BACKEND_CONTAINER_NAME = 'lendy123/backend'
    }

    stages {
        stage('Вхід у DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Білд MySQL зображення') {
            steps {
                script {
                    docker.build("mcr.microsoft.com/mssql/server:2022-latest", "-e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=Qwerty-1 -p 1433:1433 --name ${SQL_CONTAINER_NAME} --hostname sql1")
                }
            }
        }

        stage('Тегування MySQL зображення') {
            steps {
                script {
                    docker.image("mcr.microsoft.com/mssql/server:2022-latest").tag("lendy123/sql:latest")
                }
            }
        }

        stage('Пуш MySQL в DockerHub') {
            steps {
                script {
                    docker.image("lendy123/sql:latest").push()
                }
            }
        }

        stage('Білд FrontEnd зображення') {
            steps {
                script {
                    docker.build("FrontEnd/my-app/", "-t lendy123/frontend:version${BUILD_NUMBER}")
                }
            }
        }

        stage('Тегування FrontEnd зображення') {
            steps {
                script {
                    docker.image("lendy123/frontend:version${BUILD_NUMBER}").tag("lendy123/frontend:latest")
                }
            }
        }

        stage('Пуш FrontEnd в Docker Hub') {
            steps {
                script {
                    docker.image("lendy123/frontend:version${BUILD_NUMBER}").push()
                    docker.image("lendy123/frontend:latest").push()
                }
            }
        }

        stage('Білд BackEnd зображення') {
            steps {
                script {
                    docker.build("BackEnd/Amazon-clone/", "-t lendy123/backend:version${BUILD_NUMBER}")
                }
            }
        }

        stage('Тегування BackEnd зображення') {
            steps {
                script {
                    docker.image("lendy123/backend:version${BUILD_NUMBER}").tag("lendy123/backend:latest")
                }
            }
        }

        stage('Пуш BackEnd в Docker Hub') {
            steps {
                script {
                    docker.image("lendy123/backend:version${BUILD_NUMBER}").push()
                    docker.image("lendy123/backend:latest").push()
                }
            }
        }

        stage('Зупинка та видалення старого контейнера FrontEnd') {
            steps {
                script {
                    try {
                        docker.container(FRONTEND_CONTAINER_NAME).stop().remove(force: true)
                    } catch (Exception ignored) {
                        echo "Контейнер ${FRONTEND_CONTAINER_NAME} не знайдено. Продовжуємо..."
                    }
                }
            }
        }

        stage('Зупинка та видалення старого контейнера BackEnd') {
            steps {
                script {
                    try {
                        docker.container(BACKEND_CONTAINER_NAME).stop().remove(force: true)
                    } catch (Exception ignored) {
                        echo "Контейнер ${BACKEND_CONTAINER_NAME} не знайдено. Продовжуємо..."
                    }
                }
            }
        }

        stage('Зупинка та видалення старого контейнера MySQL') {
            steps {
                script {
                    try {
                        docker.container(SQL_CONTAINER_NAME).stop().remove(force: true)
                    } catch (Exception ignored) {
                        echo "Контейнер ${SQL_CONTAINER_NAME} не знайдено. Продовжуємо..."
                    }
                }
            }
        }

        stage('Чистка старих образів') {
            steps {
                script {
                    docker.image().prune(all: true, filter: "until=24h")
                }
            }
        }

        stage('Запуск Docker контейнера FrontEnd') {
            steps {
                script {
                    docker.container().run("-d -p 81:80 lendy123/frontend")
                }
            }
        }

        stage('Запуск Docker контейнера BackEnd') {
            steps {
                script {
                    docker.container().run("-d -p 5034:5034 lendy123/backend")
                }
            }
        }
    }
}

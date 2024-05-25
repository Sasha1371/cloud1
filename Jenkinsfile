pipeline {
    agent any

    environment {
        // Додаємо креденшіали для Docker
        DOCKER_CREDENTIALS_ID = 'e93a1735-85d5-4d43-9daf-a8f799888866'
        SQL_CONTAINER_NAME = 'lendy123/sql'
        FRONTEND_CONTAINER_NAME = 'lendy123/frontend'
        BACKEND_CONTAINER_NAME = 'lendy123/backend'
    }

    stages {
        stage('Вхід у Docker') {
            steps {
                script {
                    // Використовуємо креденшіали з Jenkins для входу в Docker
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                }
            }
        }
        stage('Білд MySQL зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name sql111 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest'
                }
            }
        }

        stage('Тегування MySQL зображення') {
            steps {
                script {
                    // Додаємо тег 'latest' до збудованого образу
                    sh 'docker tag mcr.microsoft.com/mssql/server:2022-latest lendy123/sql:latest'
                }
            }
        }
        stage('Пуш у MySQL в Docker Hub') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker push lendy123/sql:latest'
                }
            }
        }
        stage('Білд FrontEnd зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'cd FrontEnd/my-app/ && docker build -t lendy123/frontend:version${BUILD_NUMBER} .'
                }
            }
        }

        stage('Тегування FrontEnd зображення') {
            steps {
                script {
                    // Додаємо тег 'latest' до збудованого образу
                    sh 'docker tag lendy123/frontend:version${BUILD_NUMBER} lendy123/frontend:latest'
                }
            }
        }

        stage('Пуш у FrontEnd в Docker Hub') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker push lendy123/frontend:version${BUILD_NUMBER}'
                    sh 'docker push lendy123/frontend:latest'
                }
            }
        }

        stage('Білд BackEnd зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'cd BackEnd/Amazon-clone/ && docker build -t lendy123/backend:version${BUILD_NUMBER} .'
                }
            }
        }

        stage('Тегування BackEnd зображення') {
            steps {
                script {
                    // Додаємо тег 'latest' до збудованого образу
                    sh 'docker tag lendy123/backend:version${BUILD_NUMBER} lendy123/backend:latest'
                }
            }
        }

        stage('Пуш у BackEnd в Docker Hub') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker push lendy123/backend:version${BUILD_NUMBER}'
                    sh 'docker push lendy123/backend:latest'
                }
            }
        }

        stage('Зупинка та видалення старого контейнера FrontEnd') {
            steps {
                script {
                    // Спроба зупинити та видалити старий контейнер, якщо він існує
                    sh """
                    if [ \$(docker ps -aq -f name=^${FRONTEND_CONTAINER_NAME}\$) ]; then
                        docker stop ${FRONTEND_CONTAINER_NAME}
                        docker rm ${FRONTEND_CONTAINER_NAME}
                    else
                        echo "Контейнер ${FRONTEND_CONTAINER_NAME} не знайдено. Продовжуємо..."
                    fi
                    """
                }
            }
        }

        stage('Зупинка та видалення старого контейнера BackEnd') {
            steps {
                script {
                    // Спроба зупинити та видалити старий контейнер, якщо він існує
                    sh """
                    if [ \$(docker ps -aq -f name=^${BACKEND_CONTAINER_NAME}\$) ]; then
                        docker stop ${BACKEND_CONTAINER_NAME}
                        docker rm ${BACKEND_CONTAINER_NAME}
                    else
                        echo "Контейнер ${BACKEND_CONTAINER_NAME} не знайдено. Продовжуємо..."
                    fi
                    """
                }
            }
        }

        stage('Чистка старих образів') {
            steps {
                script {
                    // Чистимо старі образи
                    sh 'docker image prune -a --filter "until=24h" --force'
                }
            }
        }

        stage('Запуск Docker контейнера FrontEnd') {
            steps {
                script {
                    // Запускаємо Docker контейнер з новим зображенням
                    sh 'docker run -d -p 81:80 docker run -d -p 81:80 lendy123/frontend'
                }
            }
        }

        stage('Запуск Docker контейнера BackEnd') {
            steps {
                script {
                    // Запускаємо Docker контейнер з новим зображенням
                    sh 'docker run -d -p 5034:5034 lendy123/backend'
                }
            }
        }
    }
}

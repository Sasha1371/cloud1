pipeline {
    agent any

    environment {
        // Додаємо креденшіали для Docker
        DOCKER_CREDENTIALS_ID = '4d2ff1d5-c2d4-4e7b-929d-1e7504b286fb'
        CONTAINER_NAME = 'lendy123/frontend'
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
        stage('Білд FrontEnd зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'cd FrontEnd/my-app/ && docker build -t lendy123/frontend:version${BUILD_NUMBER} .'
                }
            }
        }
        stage('Тегування FrontEnd зображення & push ') {
            steps {
                script {
                    // Додаємо тег 'latest' до збудованого образу
                    sh 'docker tag lendy123/frontend:version${BUILD_NUMBER} lendy123/frontend:latest' '
                }
            }
        } 
        stage('Пуш у frontend Docker Hub') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker push lendy123/frontend:version${BUILD_NUMBER}'
                    sh 'docker push lendy123/frontend:latest
                }
            }
        }
        stage('Білд BackEnd зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'cd BackEnd/Amazon-clone/ && ls'
                }
            }
        }
        stage('Зупинка та видалення старого контейнера') {
            steps {
                script {
                    // Спроба зупинити та видалити старий контейнер, якщо він існує
                    sh """
                    if [ \$(docker ps -aq -f name=^${CONTAINER_NAME}\$) ]; then
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    else
                        echo "Контейнер ${CONTAINER_NAME} не знайдено. Продовжуємо..."
                    fi
                    """
                }
            }
        }

        stage('Чистка старих образів') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker image prune -a --filter "until=24h" --force'

                }
            }
        }
        
        
        stage('Запуск Docker контейнера') {
            steps {
                script {
                    // Запускаємо Docker контейнер з новим зображенням
                    sh 'docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name sql111 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest'
                    sh 'docker run -d -p 8081:80 --name ${CONTAINER_NAME} --health-cmd="curl --fail http://localhost:80 || exit 1" lendy123/cloudproject:version${BUILD_NUMBER}'
                    
                }
            }
        }
    }
}

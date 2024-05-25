pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'e93a1735-85d5-4d43-9daf-a8f799888866'
        CONTAINER_NAME = 'lendy123/frontend' // Может быть 'lendy123/backend' в зависимости от контекста
    }

    stages {
        stage('Вхід у Docker') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Білд FrontEnd зображення') {
            steps {
                script {
                    sh 'cd FrontEnd/my-app/ && docker build -t lendy123/frontend:version${BUILD_NUMBER} .'
                }
            }
        }

        stage('Тегування FrontEnd зображення & push ') {
            steps {
                script {
                    sh 'docker tag lendy123/frontend:version${BUILD_NUMBER} lendy123/frontend:latest'
                    sh 'docker push lendy123/frontend:version${BUILD_NUMBER}'
                    sh 'docker push lendy123/frontend:latest'
                }
            }
        }

        stage('Проверка Dockerfile') {
            steps {
                script {
                    sh 'pwd'
                    sh 'ls -a'
                    sh 'cd /home/sasha/cloud/BackEnd/Amazon-clone/ && docker build -t lendy123/backend:version${BUILD_NUMBER} .'
                }
            }
        }



        stage('Тегування BackEnd зображення ') {
            steps {
                script {
                    sh 'docker tag lendy123/backend:version${BUILD_NUMBER} lendy123/backend:latest'
                }
            }
        }

        stage('Пуш у Backend Docker Hub') {
            steps {
                script {
                    sh 'docker push lendy123/backend:version${BUILD_NUMBER}'
                    sh 'docker push lendy123/backend:latest'
                }
            }
        }

        stage('Зупинка та видалення старого контейнера') {
            steps {
                script {
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
                    sh 'docker image prune -a --filter "until=24h" --force'
                }
            }
        }

        stage('Запуск Docker контейнера') {
            steps {
                script {
                    sh 'docker run -d -p 8081:80 --name ${CONTAINER_NAME} --health-cmd="curl --fail http://localhost:80 || exit 1" lendy123/frontend:version${BUILD_NUMBER}'
                }
            }
        }
    }
}

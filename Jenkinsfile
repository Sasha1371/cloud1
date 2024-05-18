pipeline {
    agent any

    environment {
        // Додаємо креденшіали для Docker
        DOCKER_CREDENTIALS_ID = 'dockerhub'
        CONTAINER_NAME = 'lendy123_cloudproject'
    }
   
    stages {
        stage("docker login") {
            steps {
                echo " ============== docker login =================="
                withCredentials([usernamePassword(credentialsId: '4d2ff1d5-c2d4-4e7b-929d-1e7504b286fb', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def loginResult = sh(script: "docker login -u $USERNAME -p $PASSWORD", returnStatus: true)
                        if (loginResult != 0) {
                            error "Failed to log in to Docker Hub. Exit code: ${loginResult}"
                        }
                    }
                }
            }
        }
        stage('Step1') {
            steps {
                echo " ============== Build and push =================="
                sh 'docker build -t lendy123/cloudproject:version${BUILD_NUMBER} .'
                sh 'docker push lendy123/cloudproject:version${BUILD_NUMBER}'
            }
        }
        stage('Зупинка та видалення старого контейнера') {
            steps {
                script {
                    // Спроба зупинити та видалити старий контейнер, якщо він існує
                    sh """
                    if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
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
                    // Чистка старых образов
                    sh 'docker image prune -a --filter "until=5m" --force'
                }
            }
        }
        stage('Запуск Docker контейнера') {
            steps {
                script {
                    // Запуск Docker контейнера с новым изображением
                    sh 'docker run -d -p 8551:80 --name ${CONTAINER_NAME} --health-cmd="curl --fail http://localhost:80 || exit 1" lendy123/cloudproject:version${BUILD_NUMBER}'
                }
            }
        }
    }
}

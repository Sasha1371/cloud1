Ваша Jenkins Pipeline конфигурация в целом выглядит хорошо, но есть несколько проблем и улучшений, которые можно внести, чтобы исправить ошибки и улучшить выполнение.

### Проблемы и их решения

1. **Проблема с файлом `Dockerfile`**: Убедитесь, что файлы `package.json` и `package-lock.json` находятся в правильной директории относительно пути выполнения команды `docker build`.

2. **Ошибка в команде `docker run`**: У вас есть опечатка в команде `docker run`, где указано `sdocker` вместо `docker`.

3. **Использование переменной среды для имени контейнера**: Убедитесь, что переменная `CONTAINER_NAME` используется корректно.

### Исправленный скрипт

```groovy
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
```

### Дополнительные шаги:

1. **Убедитесь, что файлы `package.json` и `package-lock.json` существуют и находятся в корневой директории, откуда выполняется `docker build`.**
2. **Если они находятся в другой директории, измените команду `COPY` в вашем Dockerfile или переместите файлы.**
3. **Проверьте, что Jenkins имеет доступ к нужным директориям и файлам.**

Этот исправленный скрипт должен помочь вам успешно выполнить задачи по сборке, пушу и развертыванию вашего Docker-контейнера.

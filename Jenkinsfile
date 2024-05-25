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
            def dockerfileContent = """
                        FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
                        WORKDIR /app
                        COPY . .
                        RUN dotnet restore "ShopApi/ShopApi.csproj"

                        RUN dotnet tool install -g dotnet-ef --version 6.0

                        ENV PATH $PATH:/root/.dotnet/tools
                        RUN dotnet-ef database update --startup-project "ShopApi" --project "DAL/DAL.csproj"

                        RUN dotnet build "ShopApi/ShopApi.csproj" -c Release -o /app/build
                        FROM build AS publish

                        RUN dotnet publish "ShopApi/ShopApi.csproj" -c Release -o /app/publish
                        COPY ShopApi/images /app/publish/images
                        COPY ShopApi/comment_images /app/publish/comment_images
                        COPY ShopApi/company_images /app/publish/company_images
                        COPY ShopApi/music_images /app/publish/music_images

                        FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS final
                        WORKDIR /app
                        COPY --from=publish /app/publish .

                        EXPOSE 5034
                        ENTRYPOINT ["dotnet", "ShopApi.dll","--urls","http://0.0.0.0:5034"]
                    """
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

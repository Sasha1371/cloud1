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
                    // Використовуємо креденшіали з Jenkins для входу в Docker
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Білд Docker зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'cd FrontEnd/my-app/ && docker build -t lendy123/backend:version${BUILD_NUMBER} .'
                }
            }
        }

          stage('Тегування Docker зображення') {
            steps {
                script {
                    // Додаємо тег 'latest' до збудованого образу
                    sh 'docker tag lendy123/backend:version${BUILD_NUMBER} lendy123/backend:latest'
                }
            }
        }  
    }
}

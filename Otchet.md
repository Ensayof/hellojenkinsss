# Отчет по лабораторной работе №2
## 1. Создание и конфигурирование Jenkins Job
### 1.1 Авторизовался на Jenkins-сервере по данным полученным у преподавателя.
### 1.2 Далее новый item назвал karunos-egor, выбрал Pipeline в списке проектов и вставил измененный скрипт под свои данные.
```
pipeline {
    agent any
    
    parameters {
        string(name: 'STUDENT_NAME', defaultValue: 'Егор', description: 'Имя студента')
        string(name: 'PORT', defaultValue: '8057', description: 'Порт')
    }
    
    stages {
        stage('Удаляем старые контейнеры и образы') {
            steps {
                script {
                    // Останавливаем и удаляем контейнер с нужным именем, если он есть
                    sh "docker ps -a -q --filter name=hello-karunos-container | xargs -r docker rm -f"
                    // Удаляем образ с нужным именем, если он есть
                    sh "docker images -q student-karunos-app | xargs -r docker rmi -f"
                }
            }
        }
        stage('Выгружаем код из репозитория') {
            steps {
                git branch: 'main', url: 'https://github.com/Ensayof/hellojenkinsss.git'
            }
        }
        stage('Собираем docker image') {
            steps {
                script {
                    dockerImage = docker.build("student-karunos-app")
                }
            }
        }
        stage('Запускаем тесты в докере') {
            steps {
                script {
                    dockerImage.inside {
                        sh 'python -m unittest test_app.py'
                    }
                }
            }
        }
        stage('Запускаем докер контейнер') {
            steps {
                script {
                    sh "docker run -d --name hello-karunos-container -p ${params.PORT}:${params.PORT} -e STUDENT_NAME='${params.STUDENT_NAME}' -e PORT=${params.PORT} student-karunos-app"
                }
            }
        }
    }
}
```
## 2. В репозитории в гитхабе создал Web Hook

pipeline {
    agent any
    
    environment {
        CONTAINER_NAME = 'nginx-test'
        PORT = '9889'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Calculate MD5') {
            steps {
                script {
                    env.FILE_MD5 = sh(script: "md5sum index.html | awk '{print \$1}'", returnStdout: true).trim()
                }
            }
        }
        
        stage('Run Nginx Container') {
            steps {
                sh """
                    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 -v \$(pwd)/index.html:/usr/share/nginx/html/index.html nginx:1.24
                    sleep 5
                """
            }
        }
        
        stage('Test HTTP Response') {
            steps {
                script {
                    def response = httpRequest url: "http://localhost:${PORT}"
                    if (response.status != 200) {
                        currentBuild.result = 'FAILURE'
                        error "HTTP response status is not 200"
                    }
                }
            }
        }
        
        stage('Verify MD5') {
            steps {
                script {
                    def response = httpRequest url: "http://localhost:${PORT}"
                    // Сохраняем ответ в файл
                    writeFile file: 'response.html', text: response.content
                    // Удаляем все пробелы и переводы строк для сравнения
                    def fileMD5 = sh(script: "cat index.html | tr -d '\\n\\r ' | md5sum | awk '{print \$1}'", returnStdout: true).trim()
                    def responseMD5 = sh(script: "cat response.html | tr -d '\\n\\r ' | md5sum | awk '{print \$1}'", returnStdout: true).trim()
                    if (responseMD5 != fileMD5) {
                        currentBuild.result = 'FAILURE'
                        error "MD5 sums do not match"
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "docker rm -f ${CONTAINER_NAME} || true"
        }
        failure {
            script {
                // Здесь можно добавить отправку уведомлений в Telegram/Slack/email
                echo "Build failed! Check the logs for details."
            }
        }
    }
} 
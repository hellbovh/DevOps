pipeline {
    agent {
        label 'ansible-target'
    }
    
    stages {
        stage('Настройка прав и окружения') {
            steps {
                script {
                    echo "НАСТРОЙКА ПРАВ И ОКРУЖЕНИЯ"
                    
                    sh '''
                        echo "1. Исправляем права доступа..."
                        sudo chown -R hypocrite:hypocrite /home/hypocrite/apps/ 2>/dev/null || echo "Права уже настроены"
                        
                        echo "2. Проверяем права..."
                        ls -la /home/hypocrite/apps/
                        
                        echo "3. Останавливаем старые контейнеры..."
                        cd /home/hypocrite/apps 2>/dev/null && docker-compose down 2>/dev/null || echo "Нет контейнеров для остановки"
                        
                        echo "4. Удаляем старые контейнеры..."
                        docker rm -f frontend backend 2>/dev/null || true
                    '''
                }
            }
        }
        
        stage('Запуск приложений') {
            steps {
                script {
                    echo "ЗАПУСК ПРИЛОЖЕНИЙ"
                    
                    sh '''
                        echo "1. Переходим в директорию apps..."
                        cd /home/hypocrite/apps
                        
                        echo "2. Проверяем docker-compose.yml..."
                        if [ -f docker-compose.yml ]; then
                            echo "docker-compose.yml найден:"
                            cat docker-compose.yml
                        else
                            echo "Создаем docker-compose.yml..."
                            cat > docker-compose.yml << 'EOF'
services:
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    restart: unless-stopped

  frontend:
    build: ./frontend
    ports:
      - "8081:80"
    depends_on:
      - backend
    restart: unless-stopped
EOF
                        fi
                        
                        echo "3. Собираем и запускаем контейнеры..."
                        docker-compose build --no-cache
                        docker-compose up -d
                        
                        echo "4. Проверяем запуск..."
                        sleep 5
                        docker-compose ps
                    '''
                }
            }
            
            post {
                success {
                    script {
                        echo "Контейнеры запущены!"
                        sh '''
                            echo "ПРОВЕРКА РАБОТОСПОСОБНОСТИ"
                            echo "Даем время на полный запуск..."
                            sleep 10
                            
                            echo "1. Статус контейнеров:"
                            docker ps
                            
                            echo ""
                            echo "2. Логи backend:"
                            docker logs backend --tail 10 2>/dev/null || echo "Backend контейнер не найден"
                            
                            echo ""
                            echo "3. Логи frontend:"
                            docker logs frontend --tail 10 2>/dev/null || echo "Frontend контейнер не найден"
                            
                            echo ""
                            echo "4. Проверка доступности:"
                            echo "Backend (порт 5000):"
                            curl -s -f http://localhost:5000/read && echo "Backend работает!" || echo "Backend не отвечает"
                            echo ""
                            echo "Frontend (порт 8081):"
                            curl -s -f http://localhost:8081 && echo "Frontend работает!" || echo "Frontend не отвечает"
                            
                            echo ""
                            echo "ИНФОРМАЦИЯ ДЛЯ ПРОВЕРКИ"
                            echo "Frontend: http://localhost:8081"
                            echo "Backend API: http://localhost:5000"
                            echo "Проверить данные: curl http://localhost:5000/read"
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "ПАЙПЛАЙН ЗАВЕРШЕН"
            echo "Сборка #${BUILD_NUMBER} завершена"
        }
        success {
            echo "ЛАБОРАТОРНАЯ РАБОТА ВЫПОЛНЕНА!"
            echo "Jenkins Pipeline работает"
            echo "Контейнеры запущены через Jenkins"
            echo "Приложение доступно по ссылкам выше"
        }
        failure {
            script {
                echo "В процессе выполнения возникли ошибки"
                sh '''
                    echo "ДИАГНОСТИКА"
                    echo "1. Все контейнеры:"
                    docker ps -a
                    echo ""
                    echo "2. Сети Docker:"
                    docker network ls
                    echo ""
                    echo "3. Образы:"
                    docker images | grep -E "(frontend|backend)"
                    echo ""
                    echo "4. Файлы в apps:"
                    ls -la /home/hypocrite/apps/
                '''
            }
        }
    }
}

# Проект Kittygram - это соцсеть для котоводов, в которой пользователи могут делиться с другими пользователями фотографиями своих питомцев. 


# Установка

Склонируйте репозиторий на свой компьютер:
```
git clone git@github.com:v0vanjke/kittygram_final.git
```

Создайте файл .env и заполните его своими данными. Все необходимые переменные перечислены в файле .env.example, находящемся в корневой директории проекта.

Создайте Docker-образы:
```
cd frontend
docker build -t YOUR_DOCKERHUB_USERNAME/kittygram_frontend .
cd ../backend
docker build -t YOUR_DOCKERHUB_USERNAME/kittygram_backend .
cd ../nginx
docker build -t YOUR_DOCKERHUB_USERNAME/kittygram_gateway .
```
 
Загрузите образы на DockerHub:
```
docker push YOUR_DOCKERHUB_USERNAME/kittygram_frontend
docker push YOUR_DOCKERHUB_USERNAME/kittygram_backend
docker push YOUR_DOCKERHUB_USERNAME/kittygram_gateway
```

# Деплой на сервере

Подключитесь к удаленному серверу
```
ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME YOUR_USERNAME@SERVER_IP_ADDRESS 
```

Создайте на сервере директорию kittygram:
```
mkdir kittygram
```

Установите Docker Compose на сервер:
```
sudo apt update
sudo apt install curl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose
```

Скопируйте файлы docker-compose.production.yml в директорию kittygram/ на сервере:
```
scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml YOUR_USERNAME@SERVER_IP_ADDRESS:/home/YOUR_USERNAME/kittygram/docker-compose.production.yml

Где:

PATH_TO_SSH_KEY - путь к файлу с вашим SSH-ключом
SSH_KEY_NAME - имя файла с вашим SSH-ключом
YOUR_USERNAME - ваше имя пользователя на сервере
SERVER_IP_ADDRESS - IP-адрес вашего сервера
```

Аналогичным образом скопируйте файл .env.

Запустите Docker Compose в режиме демона:
```
sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml up -d
```

Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
```
sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py migrate
sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

Откройте конфигурационный файл Nginx в редакторе nano:
```
sudo nano /etc/nginx/sites-enabled/default
```

Измените настройки location в секции server:
```
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
}
```

Проверьте правильность конфигурации Nginx:
```
sudo nginx -t
```

Если вы получаете следующий ответ, значит, ошибок нет:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезапустите Nginx:
```
sudo service nginx reload
```


# Настройка CI/CD

Файл workflow уже написан и находится в директории:
```
kittygram/.github/workflows/main.yml
```

Для адаптации его к вашему серверу добавьте секреты в GitHub Actions:
```
DOCKER_USERNAME                # имя пользователя в DockerHub
DOCKER_PASSWORD                # пароль пользователя в DockerHub
HOST                           # IP-адрес сервера
USER                           # имя пользователя
SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/id_rsa)
SSH_PASSPHRASE                 # пароль для SSH-ключа

TELEGRAM_TO                    # ID вашего телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
TELEGRAM_TOKEN                 # токен вашего бота (получить токен можно у @BotFather, команда /token, имя бота)
```

# Технологии

Python 3.9, Django 3.2.3, djangorestframework==3.12.4, PostgreSQL 13.10

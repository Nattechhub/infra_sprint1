# Kittygram

Блог для любителей котиков! Можно добавлять фотографии, достижения, информацию, как зовут питомцев, какого они цвета и когда родились.

## Технологии
- Python 3.x
- node.js 9.x.x
- frontend: React
- backend: Django
- nginx
- gunicorn

## Установка
<br>1. Склоинровать репозиторий
```
git clone git@github.com:PARTYNEXTDOORS/infra_sprint1.git
```

<br>2. Создать и активировать виртуальное окружение
```
python3 -m venv venv
```
```
source venv/bin/activate
```

<br>3. Установить зависимости для Python
```
pip install -r requirements.txt 
```

<br>4. Перейти в папку backend и выполнить миграции
```
cd infra_sprint1/backend/
```
```
python manage.py migrate
```

<br>5. Создать суперюзера
```
python manage.py createsuperuser
```

<br>6. В файле infra_sprint1/backend/kittygram_backend/settings.py в переменную ALLOWED_HOSTS добавить локальные адреса, а также доменное имя или внешний IP (если есть). Изменить значение переменной DEBUG с True на False
```
ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'доменное имя']
```
```
DEBUG = False
```

<br>7. Установить node.js
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

<br>8. Перейти в infra_sprint1/frontend и установить зависимости для frontend-приложения. Затем скомпилировать frontend-приложение и копировать в системную директорию веб-сервера (необходимо для корректной работы статики)
```
npm i
```
```
npm run build
```
```
sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/kittygram/
```

<br>9. Для backend-приложения установить WSGI-сервер gunicorn
```
pip install gunicorn
```

<br>10. Создать файл конфигурации для автозапуска WSGi-сервера
```
sudo nano /etc/systemd/system/gunicorn_kittygram.service
```

<br>11. Внести в него следующие настройки
```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=<имя-пользователя-в-системе>  # (!) заменить на собственное
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target 
```

<br>12. Запустить созданную службу и внести её в автозапуск
```
sudo systemctl start gunicorn
```
```
sudo systemctl enable gunicorn
```

<br>13. Установить веб-сервер и запустить его
```
sudo apt install nginx -y
```
```
sudo systemctl start nginx 
```

<br>14. Открыть порты для фаерволла и активировать его
```
sudo ufw allow 'Nginx Full'
```
```
sudo ufw allow OpenSSH
```
```
sudo ufw enable
```

<br>15. Перейти в infra_sprint1/backend/, собрать статику для админки Django и копировать его в системную директорию веб-сервера (ВАЖНО! Необходимо изменить название папки со статикой во избежание конфликта имён)
```
infra_sprint1/backend/kittygram_backend/settings.py поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT

STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```
```
python manage.py collectstatic
```
```
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/
```

<br>16. Создать папку для медиафайлов в директории веб-сервера, изменить права доступа
```
cd /var/www/kittygram/
```
```
mkdir media
```
```
sudo chown -R <имя_пользователя> /var/www/kittygram/media/
```

<br>17. Перейти в файл конфигурации веб-сервера и изменить его настройки на следующие (для доступа нужны права root)
```
sudo nano /etc/nginx/sites-enabled/default 
```
```
server {

    server_name server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }

    location / {
    root    /var/www/kittygram;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```

<br>18. (Опционально) Дописать ограничения на размер файлов принимаемый nginx
```
.
.
.
    location /api/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }
.
.
.
```

<br>20. Проверить файл конфигурации веб-сервера, перезагрузить его конфиг в случае успеха
```
sudo nginx -t
```
```
sudo systemctl reload nginx
```

<br>21. (Опционально) Получить SSL-сертификат для вашего доменного имени при помощи certbot
```
sudo apt install snapd
```
```
sudo snap install core; sudo snap refresh core
```
```
sudo snap install --classic certbot
```
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
```
```
sudo certbot --nginx
```

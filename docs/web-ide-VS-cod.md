Хороший гайдик для поднятия в докере web ide
### 1. Скачиваем официальный скрипт установки
```
curl -fsSL https://get.docker.com -o get-docker.sh
```
# Запускаем его с правами суперпользователя
```
sudo sh get-docker.sh
```

### 2. Установка Docker Compose

В современных версиях Docker Compose идет как плагин. Установите его командой:
bash
```
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

### 3. Если вы не рут, то выйдате себе права

```
sudo usermod -aG docker $USER
```

### 4. Создаем Создайте файл `docker-compose.yml` :
```
version: '3.8'
services:
  code-server:
    image: codercom/code-server:latest
    container_name: code-server
    ports:
      - "8080:8080"
    environment:
      - PASSWORD=ваш_пароль_для_входа  # Укажите свой пароль
      - TZ=Europe/Moscow
    volumes:
      - ./code-data:/home/coder/project  # Здесь будут лежать ваши проекты
      - ./code-config:/home/coder/.config # Настройки и плагины
    restart: always
```
### 5. Запускаем `docker compose up`
если нет ошибок радуемся и добавляем поддомен
если есть на папки `code-data` и `code-config` выдаем права 777
команду вы и сами знаете, раз уж вы тут

### 6. Добавление правила в фаервол ufw
```
# Разрешаем порт для VS Code
sudo ufw allow 8080/tcp

# Если ты планируешь заходить в Gitea, открой порты и для неё (обычно 3000 и 22)
sudo ufw allow 3000/tcp
sudo ufw allow 22/tcp

# Перезагружаем правила, чтобы они вступили в силу
sudo ufw reload
```
### 7. Танцы с бубном для nginx
Если почему то у вас до сих пор нету nginx то ставим
```
sudo apt update
sudo apt install nginx
```
Создаем файл 
```
sudo nano /etc/nginx/sites-available/code-server
```
сам файл измените домен, на свой
```
server {
    listen 80;
    server_name ide.lord-mikrotik.ru;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Далее активируем ssl
```
# Включаем сайт
sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx

# Ставим Certbot для HTTPS
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d ide.lord-mikrotik.ru
```
Ну и на закуску добавляем в ufw
```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 8080/tcp
sudo ufw reload
```

# dns-doh
how to create doh dns 
Настройка DNS на устройстве и создание личного DNS сервера

DNS (система доменных имен) нужна для преобразования легко запоминаемых доменных имен сайтов в числовые IP-адреса, которые используют компьютеры для идентификации друг друга в сети. Благодаря DNS пользователям не нужно запоминать сложные IP-адреса, чтобы получить доступ к нужному сайту или сервису в интернете. Однако по умолчанию DNS сервера используются от провайдера интернета, что чревато утечкой данных и анализом ваших дынных, не говоря уже о блокировке ресурсов на уровне DNS. Поэтому важно вручную настроить DNS сервера на вашем устройстве.

Для начала я хочу предложить отличную подборку публичных DNS адресов, которые вы можете использовать совершенно бесплатно. Также там вы найдете DNS с автоматической фильтрацией рекламы.

Для тех кто желает настроить свой собственный DNS сервер, держите инструкцию по настройке AdGuard Home:

Регистрируем и арендуем сервер здесь, ОС выбираем Ubuntu последней версии. Подключаемся к серверу по ssh. Далее вы можете настроить быстрый и безопасный доступ по ssh ключу.

Обновите систему и установите базовые утилиты (поочередно):
```
apt update && apt upgrade -y
atp install git -y
```


Настройте файрвол UFW, введите команды (поочередно):
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp # SSH
sudo ufw allow 53/tcp # DNS TCP
sudo ufw allow 53/udp # DNS UDP
sudo ufw allow 80/tcp # HTTP (Let's Encrypt)
sudo ufw allow 3000/tcp # Adguard (временный)
sudo ufw allow 443/tcp # DoH
sudo ufw allow 853/tcp # DoT
sudo ufw allow 784/udp # DoQ
sudo ufw enable
sudo ufw status verbose
```


Проверьте, занят ли 53 порт:
```
sudo ss -tuln | grep :53
```

Если вывод пустой, то свободен, иначе:
Освободите порт 53 от systemd-resolved (по умолчанию в Ubuntu он может быть занят), введите (поочередно):
```
sudo systemctl disable systemd-resolved --now
sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
```

Скачайте и распакуйте AdGuard Home (вводим поочередно):
```
cd /opt
git clone https://github.com/admin1350/dns-doh.git
cd dns-doh
tar -xzf 'AdGuardHome_linux_amd64(1).tar.gz'
cd AdGuardHome
```


Установите AdGuard Home как сервис:
```
sudo ./AdGuardHome -s install
```


Проверьте статус сервиса:
```
sudo ./AdGuardHome -s status
```


Откройте панель управления по ссылке: http://ВАШ_IP:3000
Следуйте инструкциям, порт веб интерфейса выберите 80, а для DNS сервера 53. После установки зайдите в панель управления по адресу http://ВАШ_IP:8080

Удалите временный порт из UFW (поочередно):
```
sudo ufw delete allow 3000/tcp
sudo ufw reload
```


Настройте DNS на вашем устройстве, например: ВАШ_IP:53 (порт указывать не обязательно)

Далее (РКОМЕНДУЕТСЯ!) настройте шифрование:
Для этого шага вам понадобится домен, у которого в A записи нужно прописать IP адрес вашего сервера (это делается в ЛК регистратора домена).
Установи certbot:
```
sudo apt update
sudo apt install certbot -y
```
# Очень важно
* Если вы при настройке указали порт управления 80, то вам нужно отредактировать данный файлик `AdGuardHome/AdGuardHome.yaml'
* Изменить порт с 80 на например 81
Что бы это выгдялело так
<img width="925" height="252" alt="изображение" src="https://github.com/user-attachments/assets/5409cc05-e090-488e-b93a-d9b73c372902" />

 

Выпусти сертификат для домена:
```
Желательно сначала остановить adguard иначе не заработает systemctl stop AdGuardHome.service 
sudo certbot certonly --standalone -d doh.yourdomain.com
после запустить systemctl start AdGuardHome.service 
```



Открой в ПА Adguard Настройки > Шифрование. Поставь галочку Включить шифрование, введи свой домен, поставь галочку Автоматически перенаправлять на HTTPS, порты оставь как есть. Укажи путь к сертификату и приватный ключ:
```
/etc/letsencrypt/live/doh.yourdomain.com/fullchain.pem
/etc/letsencrypt/live/doh.yourdomain.com/privkey.pem
```
Если днс за натом, то разрешить udp 53 tcp 53 443 853

Сохрани и перейди в Инструкцию по настройке, подключи свое устройство согласно инструкции.
Про то как фильтровать рекламу и управлять списками, а также настроить DNS конфигурацию смотри в видео.
Оригинал



#приватность #гайд

3
t.me/denpiligrim_web/619

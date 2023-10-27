# Обновляем список пакетов
sudo apt update 

# Устанавливаем необходимые пакеты для работы Nextcloud и Apache
sudo apt install apache2 mariadb-server libapache2-mod-php php-gd php-mysql php-curl php-mbstring php-intl php-gmp php-bcmath php-xml php-imagick php-zip unzip

# Запускаем MySQL
sudo mysql

# Создаем пользователя и базу данных для Nextcloud
CREATE USER 'nextcloud_user'@'localhost' IDENTIFIED BY 'password';
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud_user'@'localhost';
FLUSH PRIVILEGES;

# Выходим из MySQL
quit;

# Скачиваем Nextcloud
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip

# Копируем Nextcloud в папку /var/www
sudo cp -r nextcloud /var/www

# Назначаем владельца и группу для папки Nextcloud
sudo chown -R www-data:www-data /var/www/nextcloud

# Создаем конфигурационный файл Apache
sudo nano /etc/apache2/sites-available/nextcloud.conf

# Добавляем следующий блок в конфигурационный файл
<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/
  ServerName  cloud.my-domain.com

  <Directory /var/www/nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>

# Активируем конфигурацию Nextcloud
sudo a2ensite nextcloud.conf

# Включаем необходимые модули Apache
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod env
sudo a2enmod dir
sudo a2enmod mime

# Перезапускаем службу Apache
service apache2 restart

# Устанавливаем Certbot
sudo apt-get install -y certbot

# Получаем SSL-сертификат для нашего домена
sudo certbot --nginx -d chat.my-domain.com

# Обновляем SSL-сертификаты при необходимости
sudo certbot renew --quiet

# Включаем модуль SSL для Apache
sudo a2enmod ssl
sudo a2ensite default-ssl

# Перезагружаем Apache
service apache2 reload

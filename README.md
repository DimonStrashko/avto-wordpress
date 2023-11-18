#!/bin/bash

# Оновлення пакетів
sudo yum update -y

# Встановлення та запуск HTTP-сервера (Apache)
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

# Встановлення та запуск бази даних MariaDB
sudo yum install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Налаштування бази даних MariaDB (захистити пароль root та видалити анонімних користувачів)
sudo mysql_secure_installation

# Паролі для MySQL
ROOT_PASSWORD="your_root_password"
NEW_PASSWORD="your_new_password"

# Встановлення паролю для користувача root
sudo mysqladmin -u root password "$ROOT_PASSWORD"

# Використовуємо echo для передачі відповідей під час виконання mysql_secure_installation
echo -e "\n$NEW_PASSWORD\n$NEW_PASSWORD\n\n\n\n\n" | sudo mysql_secure_installation

# Додаткові налаштування MySQL (за необхідності)
# ...

# Перезавантаження MySQL
sudo systemctl restart mysqld

# Створення бази даних та користувача для WordPress
DB_NAME="wordpressdb"
DB_USER="wordpressuser"
DB_PASSWORD="your_password"

sudo mysql -e "CREATE DATABASE $DB_NAME;"
sudo mysql -e "CREATE USER '$DB_USER'@'localhost' IDENTIFIED BY '$DB_PASSWORD';"
sudo mysql -e "GRANT ALL PRIVILEGES ON $DB_NAME.* TO '$DB_USER'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Встановлення PHP та додаткових пакетів для підтримки WordPress
sudo yum install php8.2 -y

# Перезавантаження HTTP-сервера для внесення змін
sudo systemctl restart httpd

# Завантаження та розпакування останньої версії WordPress
sudo yum install wget -y
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz -C /var/www/html/
sudo chown -R apache:apache /var/www/html/wordpress
sudo rm latest.tar.gz

# Налаштування WordPress
cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
sudo sed -i -e "s/database_name_here/$DB_NAME/" /var/www/html/wordpress/wp-config.php
sudo sed -i -e "s/username_here/$DB_USER/" /var/www/html/wordpress/wp-config.php
sudo sed -i -e "s/password_here/$DB_PASSWORD/" /var/www/html/wordpress/wp-config.php

# Додаткові налаштування для WordPress (необов'язково)
# ...

# Запуск HTTP-сервера після встановлення та налаштування WordPress
sudo systemctl restart httpd

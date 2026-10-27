# Moodle Installation (local)

## 1. Install packages

```bash
sudo apt install apache2 mysql-server php8.5 libapache2-mod-php8.5 php8.5-mysql php8.5-cli php8.5-curl php8.5-gd php8.5-mbstring php8.5-xml php8.5-zip php8.5-intl php8.5-bcmath php8.5-soap
```

## 2. Configure `php.ini`

```bash
sudo nano /etc/php/8.5/apache2/php.ini
```

```ini
max_input_vars = 5000
pcre.jit=0
```

## 3. Add the local host

```bash
sudo nano /etc/hosts
```

Add the line:

```
127.0.0.1 moodle.local
```

## 4. Create the database and user

```bash
sudo mysql
```

```sql
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 5. Configure virtual host

```bash
sudo nano /etc/apache2/sites-available/moodle.conf 
```

```conf
<VirtualHost *:80>
        ServerName moodle.local
        DocumentRoot /var/www/moodle/public
        <Directory /var/www/moodle/public>
            AllowOverride None
            Require all granted
#           FallbackResource /r.php
        </Directory>
</VirtualHost>
```

```bash
$ cd /etc/apache2/sites-available/
$ sudo a2dissite 000-default
$ sudo a2ensite moodle
$ sudo systemctl restart apache2
```

## 6. Permissions

```bash
sudo chown -R www-data:www-data /var/www/moodle
sudo chown -R www-data:www-data /var/www/moodledata
```

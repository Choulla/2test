# Projet de fin d'étude sous magento 2.3.7 : Trust-Parcel 

## Index

1. Pré-requis
2. Préparation du serveur web
3. Préparation du serveur mysql
4. Installation des dépendances
5. Autre configuration
6. Vérifier l'instalation

## 1. Pré-requis

* [Magento 2](https://devdocs.magento.com/guides/v2.3/install-gde/prereq/prereq-overview.html)

Php version 7.4.x  pour vérifier: php -v

Apache 2 version 2.4. 

Mysql version 5.7 min.

Composer 2

## 2. Préparation du serveur web

### 2.1 Configuration du serveur web
Utilisation d'un serveur local apache 2
enabling module rewrite :
- sudo a2enmod rewrite

Disabling mode php on apache2:
- sudo a2dismod php7.4

## 2.2 Installation des extensions de php:
```
sudo apt-get install php7.4 php7.4-fpm libapache2-mod-php7.4 php7.4-common php7.4-gd php7.4-mysql php7.4-curl \
php7.4-intl php7.4-xsl php7.4-mbstring php7.4-zip php7.4-bcmath php7.4-soap php-xdebug php-imagick \
php7.4-fpm php7.4-cli php7.4-curl php7.4-mysql php7.4-sqlite3 php7.4-gd php7.4-xml php7.4-mcrypt \
php7.4-mbstring php7.4-iconv
```
N'oubliez pas de vérifier si ces extensions aussi existent : php-imagick, php-xdebug , php7.4-common .

### 2.3 Création d'un virtual host

#### 2.3.1 création d'un fichier de config
sudo gedit /etc/apache2/sites-available/trust-parcel.conf

#### 2.3.2 copie le texte suivant dans le fichier crée

```
<VirtualHost *:80>
    ServerName trust-parcel.fr
    DocumentRoot "/var/www/html/magento2pfe"

    # Sources
    <Directory "/var/www/html/magento2pfe">
        Options Indexes FollowSymLinks MultiViews Includes
        AllowOverride all
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
#    <FilesMatch \.php$>
#    SetHandler "proxy:unix:/run/php/php7.4-fpm.sock|fcgi://localhost"
#    </FilesMatch>
    # Logs
    ErrorLog "/var/log/apache2/trust-parcel.error_log"
    CustomLog "/var/log/apache2/trust-parcel.access_log" common
</VirtualHost>
```
#### 2.3.3 créer un lien symbolique 

sudo ln -s /etc/apache2/sites-available/trust-parcel.conf /etc/apache2/sites-enabled

#### 2.3.4 Vérifier la configuration
apachectl configtest ==> résultat doit être ok
```
Syntax OK

```
Redémarrer le serveur apache2

```
sudo service apache2 restart
```

### 2.4 Ajout de l'url au fichier hosts
sudo gedit /etc/hosts
```
127.0.0.1	trust-parcel.fr

```

## 3. Préparation du serveur mysql

### 3.1 création et Configuration de la base de données

garder le même nom du base des données 

```
create database magento2_pfe;

create user magento IDENTIFIED BY 'magento';

GRANT ALL PRIVILEGES ON magento2_pfe.* TO magento@localhost;

flush privileges;
```

### 3.2 import de la base de données

mysql -umagento -pmagento magento2_pfe < dump/exportdb.sql

## 4. Installation des dépendances

### 4.1 Création des keys sous magento marketplace:

https://devdocs.magento.com/guides/v2.2/install-gde/prereq/connect-auth.html

```
Use the Public key as your username and the Private key as your password.
```
Il faut créer le public key et le private key pour les utilisées  aprés dans l'étape suivante.

### 4.2 Installation de dépendances:

```
composer install	

```
PS: il ne faut jamais utiliser sudo avec composer, cela va entrainer des problèmes de permissions après,

Il faut activer le mode developer et compiler avec les commandes suivantes:

```
chmod u+x bin/magento
sudo chown -R :www-data .

bin/magento deploy:mode:set developer
bin/magento cache:flush 
bin/magento cache:clean 
bin/magento app:config:import -n
bin/magento setup:upgrade
bin/magento indexer:reindex
```

### 4.3 Modification de permissions:

```
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
```

### 5 Autre configuration:

```
select * from core_config_data where path like '%url%';
update core_config_data set value='http://trust-parcel.fr' where config_id=4;
bin/magento cache:flush && bin/magento cache:clean

sudo service php7.4-fpm stop
sudo a2enmod php7.4
service apache2 restart


```
### 6 Vérifier l'installation

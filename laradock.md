1.docker_laravel直下で下記を実行
git clone https://github.com/Laradock/laradock.git

2.laradock直下で下記を実行
cp .env.example .env

3.envの下記を変更
PHP_VERSION=8.2
MYSQL_VERSION=5.7
APP_CODE_PATH_HOST=../test/

MySQLが5.7の場合は下記を修正しないと起動しない
DATA_PATH_HOST=.laradock/data

4.laradock直下で下記を実行
docker-compose up -d apache2 mysql phpmyadmin redis workspace

http://localhost/

5.下記でworkspaceに入る
winpty docker exec -it プロセスID bash（winptyはWindowsのみ必要）
プロセスIDはdocker psで調べる

6.workspaceに入り、/var/www直下で下記を実行
composer create-project laravel/laravel test "10.*"

7.laradock直下のapache2/sitesで下記を実行
cp sample.conf.example sample.conf

8.sample.confを下記のように変更する
<VirtualHost *:80>
  ServerName localhost
  DocumentRoot /var/www/test/public/
  Options Indexes FollowSymLinks

  <Directory "/var/www/test/public/">
    AllowOverride All
    <IfVersion < 2.4>
      Allow from all
    </IfVersion>
    <IfVersion >= 2.4>
      Require all granted
    </IfVersion>
  </Directory>

</VirtualHost>


8.停止して、再度起動
docker-compose stop
docker-compose up -d --build apache2 mysql phpmyadmin redis workspace


9.エラーが出る
The stream or file "/var/www/test/storage/logs/laravel.log" could not be opened in append mode: Failed to open stream: Permission denied The exception occurred while attempting to log

workspaceに入り、/var/www/test直下で下記を実行
chmod -R 777 storage

10.C:\docker_laravel\test\test\config\app.phpを修正
'timezone' => 'Asia/Tokyo',
'locale' => 'ja',

11.C:\docker_laravel\test\test\.envを修正
APP_DEBUG=true

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=default
DB_USERNAME=root
DB_PASSWORD=root

12.workspaceに入り、/var/www/test直下で下記を実行
composer require barryvdh/laravel-debugbar
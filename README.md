# **Лаборатораня работа №5 "Запуск сайта в контейнере"**

## Цель работы

Выполнив данную работу студент сможет подготовить образ контейнера для запуска веб-сайта на базе Apache HTTP Server + PHP (mod_php) + MariaDB.

### Задание

Создать Dockerfile для сборки образа контейнера, который будет содержать веб-сайт на базе Apache HTTP Server + PHP (mod_php) + MariaDB. База данных MariaDB должна храниться в монтируемом томе. Сервер должен быть доступен по порту 8000.

Установить сайт WordPress. Проверить работоспособность сайта.

### Подготовка

Для выполнения данной работы необходимо иметь установленный на компьютере Docker.

Для выполнения работы необходимо иметь опыт выполнения лабораторной работы №4.

### Описание выполнения работы

1. Создаем репозиторий `containers05` и клонируем его на компьютер. Для этого используем команду ```git clone``` и переходим в нашу склонированную папку.
2. Создаём в папке containers04 папку files, а также
    - папку files/apache2 - для файлов конфигурации apache2;
    - папку files/php - для файлов конфигурации php;
    - папку files/mariadb - для файлов конфигурации mariadb.

3. Создаём в папке containers05 файл Dockerfile со следующим содержимым:

  ```dockerfile
    # create from debian image
  FROM debian:latest

  # install apache2, php, mod_php for apache2, php-mysql and mariadb
  RUN apt-get update && \
      apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
      apt-get clean
  ```

4. Строим образ контейнера с именем apache2-php-mariadb, с помощью команда `docker build -t apache2-php-mariadb .`
![Img-1](https://imgur.com/jSqbVdw.png)
Создайте контейнер apache2-php-mariadb из образа apache2-php-mariadb и запустите его в фоновом режиме с командой запуска bash.
![Img-2](https://imgur.com/4q4GGgQ.png)
5. Копируем из контейнера файлы конфигурации ```apache2, php, mariadb``` в папку ```files/``` на компьютере. Для этого используем команды:

```shell
docker cp apache2-php-mariadb:/etc/apache2/sites-available/000-default.conf files/apache2/
docker cp apache2-php-mariadb:/etc/apache2/apache2.conf files/apache2/
docker cp apache2-php-mariadb:/etc/php/8.2/apache2/php.ini files/php/
docker cp apache2-php-mariadb:/etc/mysql/mariadb.conf.d/50-server.cnf files/mariadb/
```
![Img-3](https://imgur.com/KGfvJlM.png)

6. После выполнения команд в папке files/ должны появиться файлы конфигурации apache2, php, mariadb. Проверяем их наличие.
![Img-4](https://imgur.com/T2xrBEB.png)
7. Остановите и удалите контейнер apache2-php-mariadb.
![Img-5](https://imgur.com/AIOOc0m.png)


### Настраиваем конфигурационные файлы

1. Конфигурационный файл ```apache2```:

- Открываем файл ```files/apache2/000-default.conf```, найдите строку ```#ServerName www.example.com``` и замените её на ```ServerName localhost```.
- Ищем строку ```ServerAdmin webmaster@localhost``` и меняем в ней почтовый адрес на свой.
- После строки ```DocumentRoot /var/www/html``` добавляем следующие строки: ```DirectoryIndex index.php index.html```
- Сохраняем файл и закрываем.
- В конце файла ```files/apache2/apache2.conf``` добавьте следующую строку: ```ServerName localhost```

2. Конфигурационный файл ```php```:

- Открывем файл ```files/php/php.ini```, ищем строку ```;error_log = php_errors.log``` и меняем её на ```error_log = /var/log/php_errors.log```.
- Настраиваем параметры ```memory_limit, upload_max_filesize, post_max_size``` и ```max_execution_time``` следующим образом:

   ```shell
   memory_limit = 128M
   upload_max_filesize = 128M
   post_max_size = 128M
   max_execution_time = 120
   ```

- Сохраняем файл и закрываем его.

3. Конфигурационный файл ```mariadb```:

- Открывем файл ```files/mariadb/50-server.cnf```, ищем строку ```#log_error = /var/log/mysql/error.log``` и раскомментиуем её.
- Сохраняем файл и закрываем его.

4. Создаем скрипт запуска:
-Создаем в папке ```files``` папку ```supervisor``` и файл ```supervisord.conf``` со следующим содержимым:

```shell
[supervisord]
nodaemon=true
logfile=/dev/null
user=root

# apache2
[program:apache2]
command=/usr/sbin/apache2ctl -D FOREGROUND
autostart=true
autorestart=true
startretries=3
stderr_logfile=/proc/self/fd/2
user=root

# mariadb
[program:mariadb]
command=/usr/sbin/mariadbd --user=mysql
autostart=true
autorestart=true
startretries=3
stderr_logfile=/proc/self/fd/2
user=mysql
```

5. Создаем ```Dockerfile```:  
Открываем файл Dockerfile и добавляем в него следующие строки:

- после инструкции FROM ... добавляем монтирование томов:

```shell
# mount volume for mysql data
VOLUME /var/lib/mysql

# mount volume for logs
VOLUME /var/log
```

- в инструкции RUN ... добавляем установку пакета supervisor.
- после инструкции RUN ... добавляем копирование и распаковку сайта WordPress:

```shell
# add wordpress files to /var/www/html
ADD https://wordpress.org/latest.tar.gz /var/www/html/
```

- после копирования файлов WordPress добавляем копирование конфигурационных файлов apache2, php, mariadb, а также скрипта запуска:

```shell
# copy the configuration file for apache2 from files/ directory
COPY files/apache2/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY files/apache2/apache2.conf /etc/apache2/apache2.conf

# copy the configuration file for php from files/ directory
COPY files/php/php.ini /etc/php/8.2/apache2/php.ini

# copy the configuration file for mysql from files/ directory
COPY files/mariadb/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf

# copy the supervisor configuration file
COPY files/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
```

для функционирования mariadb создайте папку /var/run/mysqld и установите права на неё:

```shell
# create mysql socket directory
RUN mkdir /var/run/mysqld && chown mysql:mysql /var/run/mysqld
```

- открываем порт 80.
  `# Expose port 80 for Apache
EXPOSE 80
`
- добавляем команду запуска *supervisord*:

```shell
# start supervisor
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]
```

6. Собераем Docker-образ `docker build -t apache2-php-mariadb` .
![Img-6](https://imgur.com/lI7itlb.png)
![Img-7](https://imgur.com/uoIKL4q.png)
Запускаем контейнер из образа `docker run -d --name apache2-php-mariadb -p 80:80 apache2-php-mariadb`.
![Img-8](https://imgur.com/AT7u91d.png)

### Создаем базу данных и пользователя

Создаем базу данных *wordpress* и пользователя *wordpress* с паролем *wordpress* в контейнере *apache2-php-mariadb*. для этого выполняем команду: `docker exec -it apache2-php-mariadb mysql -u root -p `

```sql
CREATE DATABASE wordpress;
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'wordpress';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
![Img-9](https://imgur.com/NKSNsSU.png)

#### Создаем файл конфигурации WordPress

1. Открываем в браузере сайт WordPress по адресу *<http://localhost/8080>*, указываем параметры для подключения, которые были применены при создании базы данных.  
![Img-10](https://imgur.com/9gOliwq.png)

2. Копируем содержимое файла конфигурации в файл *files/wp-config.php* на компьютере ` docker cp apache2-php-mariadb:/var/www/html/wp-config.php ./files/wp-config.php`.
   
![Img-11](https://imgur.com/UtLsKyo.png)
![Img-12](https://imgur.com/Ak3sVc7.png)
![Img-13](https://imgur.com/pX46ZWs.png)

### Добавим файл конфигурации WordPress в Dockerfile

Добавляем в файл Dockerfile следующие строки:

 ```shell
# copy the configuration file for wordpress from files/ directory
COPY files/wp-config.php /var/www/html/wordpress/wp-config.php
```

### Запуск и тестирование

Пересобираем образ контейнера с именем *apache2-php-mariadb* и запускаем контейнер *apache2-php-mariadb* из образа *apache2-php-mariadb.* Проверяем работу сайта WordPress.
![Img-14](https://imgur.com/NJctznp.png)
![Img-15](https://imgur.com/qcz91RK.png)

### Вывод

В процессе работы был развернут контейнер с именем apache2-php-mariadb, содержащий среду для работы с WordPress, включая веб-сервер Apache, PHP и базу данных MariaDB. Для обеспечения сохранности данных был настроен монтируемый том для MariaDB. Кроме того, выполнена настройка Apache с поддержкой PHP, установка и конфигурация WordPress, а также запуск веб-сайта на его основе. 

### Ответы на вопросы

1. Были изменены следующие файлы конфигурации:

- ```files/apache2/000-default.conf```– настройка виртуального хоста Apache.
- ```files/apache2/apache2.conf``` – общие настройки Apache.
- ```files/php/php.ini``` – конфигурация PHP.
- ```files/mariadb/50-server.cnf``` – конфигурация MariaDB.
- ```files/supervisor/supervisord.conf``` – настройки процесс-менеджера Supervisor.
- ```files/wp-config.php``` – конфигурационный файл WordPress.

2. Инструкция `DirectoryIndex` указывает файл, который Apache загружает по умолчанию при обращении к каталогу. В лабораторной работе она настроена на `index.php` `index.html`, что позволяет WordPress работать корректно.

3. Файл `wp-config.php` нужен для настройки WordPress. Основные функции:
- Подключение к базе данных – содержит имя БД, пользователя, пароль, хост.
- Настройки безопасности – секретные ключи для защиты сессий.
- Префиксы таблиц – позволяет изменять структуру БД и повышает безопасность.
- Конфигурация отладки – включает/выключает режим отладки (WP_DEBUG).
- Дополнительные настройки – управление кешем, языком, путями и API.

4. Параметр ```post_max_size``` определяет максимальный размер данных, которые можно отправить в одном *POST*-запросе.
5. Недостатки:

- **Все сервисы в одном контейнере** – лучше разделить Apache+PHP и MariaDB с помощью `docker-compose`.
-  **Хранение паролей в образе** – безопаснее передавать их через переменные окружения.
-   **Большой размер образа** – вместо `debian:latest` лучше использовать `debian-slim` или `alpine`. 🚀

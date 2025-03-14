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
  1. Создаем репозиторий `containers05` и клонируем его на компьютер. Для этого используем команду ```git clone ``` и переходим в нашу склонированную папку.
  2. Создаём в папке containers04 папку files, а также
- папку files/apache2 - для файлов конфигурации apache2;
- папку files/php - для файлов конфигурации php;
- папку files/mariadb - для файлов конфигурации mariadb.
3. Создаём в папке containers05 файл Dockerfile со следующим содержимым:
```sql
  # create from debian image
FROM debian:latest

# install apache2, php, mod_php for apache2, php-mysql and mariadb
RUN apt-get update && \
    apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
    apt-get clean
```
4. Строим образ контейнера с именем apache2-php-mariadb, с помощью команда ```docker build -t apache2-php-mariadb .```

 



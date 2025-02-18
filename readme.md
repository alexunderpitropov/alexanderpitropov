# Лабораторная работа №1.Виртуальный сервер

**Питропов Александр,группа I2302**  
**Дата выполнения:15.02.2025** 

# Описание задачи

Цель данной лабораторной работы – ознакомиться с виртуализацией операционных систем, установкой и настройкой виртуального HTTP-сервера. Для этого мы развернем Debian в QEMU, настроим LAMP-стек, установим PhpMyAdmin и CMS Drupal, а также сконфигурируем виртуальные хосты Apache.

# Выполнение работы

# Подготовка к лабороторной работе

Скачиваю MSYS2,после установки создаю папку lab01 при помощи команды `mkdir lab01`.Далее в этой папке я создаю еще одну папку `dvd`,а так же файл `readme.md`.
На сайте Debian,скачиваю 64-bit PC DVD-1 iso.Потом я переиминовываю скачанный файл в `debian.iso` при помощью `mv`.Так же я переместил этот файл в папку `dvd`.

Далее я скачиваю  QEMU

![image](https://i.imgur.com/AWbFF9M.png)

После этого QEMU успешно установлен

# Установка OC Debian

В консоли в папке lab01 создаю образ диска для виртуальной машины размером 8gb,формата qcow2 при помощи утилиты qemu-img,используя команду `qemu-img create -f qcow2 debian.qcow2 8G`

![image](https://i.imgur.com/cKOmJza.png)

Начинаю установку OC Debian командой `qemu-system-x86_64 -hda debian.qcow2 -cdrom dvd/debian.iso -boot d -m 2G`

![image](https://i.imgur.com/r5WvNPb.jpeg)

Вот результат:

Я выбрал следующие параметры при установке Debian:
Имя компьютера: `debian`
Хостовое имя: `debian.localhost`
Имя пользователя: `user`
Пароль пользователя: `password`
Во время установки на этапе Software Selection выбрал для установки standart system utilities,без графической оболочки и рабочего стола.

# Запуск виртуальной машины

Для запуска я использовал следующую команду:
`qemu-system-x86_64 -hda debian.qcow2 -m 2G -smp 2 \-device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::1080-:80,hostfwd=tcp::1022-:22`

# Установка LAMP

Переключаюсь на суперпользователя при помощи команды `su`

![image](https://i.imgur.com/Yusr9u6.png)
**До установки следующих пакетов я скачал следующие репозитории,чтобы можно было подключиться к серверу**

![image](https://i.imgur.com/AzQx1rC.png)

**Выполнил следующие команды:**
apt update -y`

![image](https://i.imgur.com/lkkIF3s.png)

`apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client unzip`

![image](https://i.imgur.com/AJcwYa0.png)

**Вот краткое описание установленных пакетов и их назначения:**
apache2 — веб-сервер Apache
php — интерпретатор PHP для выполнения скриптов
libapache2-mod-php — модуль Apache, обеспечивающий поддержку PHP
php-mysql — расширение PHP для взаимодействия с MySQL/MariaDB
mariadb-server — сервер базы данных MariaDB (форк MySQL)
mariadb-client — клиент для работы с сервером MariaDB
unzip — утилита для извлечения файлов из ZIP-архивов

# Установка PhpMyAdmin и CMS Drupal

**PhpMyAdmin и CMS Drupal скачиваю с помощью следующих команд:**
`wget https://files.phpmyadmin.net/phpMyAdmin/5.2.2/phpMyAdmin-5.2.2-all-languages.zip`

![image](https://i.imgur.com/VNLcsUW.png)

`wget https://ftp.drupal.org/files/projects/drupal-11.1.1.zip`

![image](https://i.imgur.com/RgYrixn.png)

**Проверка на наличие файлов.**

![image](https://i.imgur.com/5jmo6m8.png)

**Распаковка и перемещение файлов:**

`mkdir /var/www`
`unzip phpMyAdmin-5.2.2-all-languages.zip`
`mv phpMyAdmin-5.2.2-all-languages /var/www/phpmyadmin`
`unzip drupal-10.0.5.zip`
`mv drupal-10.0.5 /var/www/drupal`

# Настройка БД

Создаем через командную строку для CMS базу данных drupal_db и пользователя с именем user

**Прописываю команду mysql -u root,нужна для подключения к серверу MariaDB**

**Далее прописываю следующие команды:**

![image](https://i.imgur.com/Nl0DMJL.png)

# Настройка виртуальных хостов Apache

**Создаю файлы конфигурации**

**PhpMyAdmin:**

`nano /etc/apache2/sites-available/01-phpmyadmin.conf`

**Содержимое:**
```
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot "/var/www/phpmyadmin"
        ServerName phpmyadmin.localhost
        ServerAlias www.phpmyadmin.localhost
        ErrorLog "/var/log/apache2/phpmyadmin.localhost-error.log"
        CustomLog "/var/log/apache2/phpmyadmin.localhost-access.log" common
    </VirtualHost>
```

![image](https://i.imgur.com/3djveez.png)

**CMS Drupal:**

`nano /etc/apache2/sites-available/02-drupal.conf`

**Содержимое:**
```
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot "/var/www/drupal"
        ServerName drupal.localhost
        ServerAlias www.drupal.localhost
        ErrorLog "/var/log/apache2/drupal.localhost-error.log"
       CustomLog "/var/log/apache2/drupal.localhost-access.log" common
   </VirtualHost>
```
![image](https://i.imgur.com/mpaPa2M.png)

**Зарегестрирую конфигурацию:**
`/usr/sbin/a2ensite 01-phpmyadmin`

`/usr/sbin/a2ensite 02-drupal`

![image](https://i.imgur.com/hBZrJiw.png)

**Добавляю записи в /etc/hosts**

`127.0.0.1 phpmyadmin.localhost`

`127.0.0.1 drupal.localhost`

![image](https://i.imgur.com/k5D8lXO.png)

# Запуск и тестирование

Проверяю версию системы:

`uname -a`

Перезапускаю Apache:

`systemctl restart apache2`

Так же проверяю доступность сайтов в браузере

`http://drupal.localhost:1080`

`http://phpmyadmin.localhost:1080`

![image](https://i.imgur.com/7zrtULP.png)

![image](https://i.imgur.com/axWup46.png)

![image](https://i.imgur.com/2AxcqDW.png)

![image](https://i.imgur.com/kxT7gHh.jpeg)

На этом этапе я заметил, что версия PHP интерпретатора не вызвала проблем, однако для корректной работы потребовалась установка некоторых пакетов.

Сначала я решил проверить, какие пакеты уже установлены, используя следующую команду в терминале:

Я обновил списки пакетом:

`apt update`

Далее я установил недостающие пакеты:

`apt install php-xml php-gd`

![image](https://i.imgur.com/cA6DVL6.png)

Далее я перезагрузил Apache

![image](https://i.imgur.com/Wq9Ix21.png)

После данных манипуляций ошибки,которые были связанны с PHP пропали.

Следующая ошибка была связана с File System. Для ее устранения я использовал следующие команды команд:

`chown -R www-data /var/www/drupal`

Данная команда передает права на папку /var/www/drupal и все её содержимое пользователю www-data, чтобы веб-сервер мог работать с файлами Drupal.

Далее я вывожу информацию о файлах и папках внутри /var/www/drupal:

`ls -l /var/www/drupal`

![image](https://i.imgur.com/oeFzyMP.png)

**Ошибок больше нет**

![image](https://i.imgur.com/EK3C3CB.png)

![image](https://i.imgur.com/eFL4EvM.png)

![image](https://i.imgur.com/gexAxiU.png)

![image](https://i.imgur.com/0sf167b.png)

![image](https://i.imgur.com/n24Xw73.jpeg)

# Ответы на вопросы

1.Скачивание файла с помощью wget в консоли

Используйте команду:

`wget` <URL>

Например, чтобы скачать файл file.zip:

`wget https://example.com/file.zip`

Можно добавить флаг -O, чтобы задать имя сохраняемого файла:

`wget -O myfile.zip https://example.com/file.zip`

Если нужно загрузить файл в фоновом режиме, используйте -b:

`wget -b https://example.com/file.zip`

2.Зачем создавать для каждого сайта свою базу и пользователя?

Безопасность: Если один сайт будет скомпрометирован, другие базы данных не пострадают.

Изоляция данных: Разные сайты могут иметь разные схемы данных и требования.

Управление доступом: Можно ограничить права пользователей в зависимости от нужд сайта.

Производительность: Разделение БД помогает лучше распределять нагрузку.

3.Как поменять порт управления БД на 1234?

Например, для MySQL/MariaDB:

Открыть конфигурационный файл (обычно `/etc/mysql/my.cnf или /etc/my.cnf`).

Найти секцию [mysqld] и изменить строку:

port=1234

Перезапустить MySQL:

`systemctl restart mysql`

Для PostgreSQL:

Изменить `postgresql.conf` (обычно `/etc/postgresql/XX/main/postgresql.conf`):

port = 1234

Перезапустить PostgreSQL:

`systemctl restart postgresql`

4.Преимущества виртуализации

Экономия ресурсов: Несколько виртуальных машин (ВМ) на одном физическом сервере.

Изоляция: Проблемы одной ВМ не влияют на другие.

Гибкость: Можно легко развертывать, клонировать и перемещать ВМ.

Масштабируемость: Быстрое развертывание новых серверов.

Снижение затрат: Меньше физического оборудования → ниже затраты.

5.Зачем устанавливать время / временную зону на сервере?

Корректная работа логов: Временные метки должны соответствовать реальному времени.

Синхронизация процессов: Некоторые сервисы (например, базы данных) зависят от точного времени.

Безопасность: Сертификаты SSL/TLS и механизмы аутентификации часто используют временные метки.

Автоматизация задач: Планировщики задач (cron) работают с учётом времени.

6.Сколько места занимает установленная ОС (виртуальный диск)?

У меня ровно 2gb

7.Рекомендации по разбиению диска для серверов
Рекомендуемая структура разбиения (пример для Linux-сервера):

/boot – 512 МБ–1 ГБ (для загрузочных файлов).

/ (root) – 10-20 ГБ (основная файловая система).

swap – 1-2×RAM (если сервер использует swap).

/home – зависит от количества пользователей.

/var – 10+ ГБ (для логов, кеша, баз данных).

/tmp – 2-10 ГБ (для временных файлов).

/data – (если есть база данных или хранилище файлов).

Почему так делают?

Изоляция критичных данных: Отдельный /var предотвращает переполнение корневого раздела логами.
Безопасность: /home и /tmp можно монтировать с ограничениями (noexec, nodev).
Гибкость: Легче управлять местом и бэкапами.

# Вывод

В ходе данной лабораторной работы я освоил навыки установки и настройки операционной системы на хостовой машине, изучил все ключевые аспекты её функционирования. Также я провел проверку работоспособности системы и приобрел ценные знания, которые обязательно пригодятся мне в будущем при работе с хостовой машиной и аналогичными системами.

# Библиография

Официальный сайт Debian. https://www.debian.org

Официальный сайт MYSYS2 https://www.msys2.org/

Официальная документация QEMU. https://www.qemu.org/documentation

Гайд по обновлению php https://php.watch/articles/php-8.3-install-upgrade-on-debian-ubuntu

ChatGPT https://chatgpt.com/
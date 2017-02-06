Автоматический запуск droplets на DigitalOcean, установка Docker и создание контейнеров с phppgadmin и postgresql.
===========

- Контейнеры базируются на Debian [Debian official repository](https://index.docker.io/_/debian/)
- droplets на Ubuntu 16.04

### Создание и запуск droplets
ansible-do-api/digitalocean.yml

    droplets:
    - manager1
    #- worker1
    #- worker2

Указываем имена создаваемых машин, в нашем случае достаточно одной. 
playbook автоматически проверяет наличие ssh ключей и в случае их отсутствия генерирует новые. 
Используя API DigitalOcean добавляет эти ключи в аккаунт.
Создаёт машины по списку с заданными параметрами, для дальнейшей авторизации прописывается ssh ключ
Для запуска:
	$ cd ansible-do-api && ansible-playbook digitalocean.yml

Список IP для вновь созданных droplets попадает в ansible-docker/newhosts

### Установка docker в droplets

ansible-docker/hosts

[mydockerhosts]

[manager]

[workers]

вписываем IP хостов в нужные разделы в зависимости от роли, в нашем случае будет достаточно добавить IP в раздел 

[mydockerhosts]

  $ cd ansible-docker && ansible-playbook 1-installdocker.yml

происходит установка docker и docker-compose на все droplets (если будет использоваться swarm, то можно docker-compose ставить только на manager ноду)

  $ ansible-playbook 2-phppg+pgsql.yml

Распаковывается архив ansible-docker/phppgadmin.tar.gz с конфигурацией для docker-compose на ноду и выполняются команды по сборке и запуску контейнеров уже средствами docker-compose

### Для PostgreSQL используется image orchardup/postgresql

Переменные для создания начальных параметров PostgreSQL хранятся в ansible-docker/phppgadmin/pgsql-variables.env

POSTGRESQL_USER=test

POSTGRESQL_PASS=oe9jaacZLb33R9pN

POSTGRESQL_DB=test



### Apache и Postgres переменные окружения

Apache и Postgres будут запущены со следующими переменными.

ENV APACHE_RUN_USER www-data

ENV APACHE_RUN_GROUP www-data

ENV APACHE_LOG_DIR /var/log/apache2

ENV APACHE_PID_FILE /var/run/apache2.pid

ENV APACHE_RUN_DIR /var/run/apache2

ENV APACHE_LOCK_DIR /var/lock/apache2

ENV APACHE_SERVERADMIN admin@localhost

ENV APACHE_SERVERNAME localhost

ENV POSTGRES_DEFAULTDB test

ENV POSTGRES_HOST postgresql

ENV POSTGRES_PORT 5432

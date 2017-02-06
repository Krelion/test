Автоматический запуск droplets на DigitalOcean, установка Docker и создание контейнеров с phppgadmin и postgresql.
===========

- Контейнеры базируются на Debian [Debian official repository](https://index.docker.io/_/debian/)
- droplets на Ubuntu 16.04
- ansible 2.2.1.0

### Создание и запуск droplets
ansible-do-api/digitalocean.yml

Здесь указываем свой API ключ DigitalOcean

    vars:
      do_token: your-do-token

Указываем имена создаваемых машин, в нашем случае достаточно одной. 

    droplets:
    - manager1
    #- worker1
    #- worker2

playbook автоматически проверяет наличие ssh ключей и в случае их отсутствия генерирует новые. 
Используя API DigitalOcean добавляет эти ключи в аккаунт.
Создаёт машины по списку с заданными параметрами. 
Для дальнейшей авторизации прописывается ssh ключ.

$ cd ansible-do-api && ansible-playbook digitalocean.yml

Список IP для вновь созданных droplets попадает в ansible-docker/newhosts

### Установка docker в droplets

Вписываем IP хостов в нужные разделы в зависимости от роли, в нашем случае будет достаточно добавить IP в раздел 

[mydockerhosts]

файл ansible-docker/hosts

Запускаем установку docker и docker-compose на все droplets (если будет использоваться swarm, то можно docker-compose ставить только на manager ноду)

  $ cd ansible-docker && ansible-playbook 1-installdocker.yml

Автоматически распаковывается архив ansible-docker/phppgadmin.tar.gz с конфигурацией для docker-compose на ноду и выполняются команды по сборке и запуску контейнеров уже средствами docker-compose

  $ ansible-playbook 2-phppg+pgsql.yml


### Для PostgreSQL используется image orchardup/postgresql с нужными нам параметрами

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

### Дополнительные playbooks для массового развертывания swarm ansible-docker/additional/

Переносим файлы из ansible-docker/additional/ в ansible-docker

Для создания ноды с менеджером

$ ansible-playbook -vv managerswarm.yml

Для подключения нод с воркерами

$ ansible-playbook workertoswarm.yml

Предварительно нужно прописать token и адрес менеджера, полученные после завершения работы первой команды, в переменные в файле 

workertoswarm.yml

    hosts:
      - workers
    vars:
      ansible_python_interpreter: "/usr/bin/python3"
      token: "SWMTKN-1-44z5u2rjv6kd5b8dyan1copb702ntdwvm13jbslesmtchp6y41-cmrv2dzhunckqax0mueet2sz4"
      manageraddres: "128.199.34.37:2377"

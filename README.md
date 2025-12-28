DOCKER / DOCKERFILE / DOCKER COMPOSE — CHEATSHEET
(ASCII / PRE-FORMATTED README — НЕ ПЛЫВЁТ В GITHUB)
===============================================================================

Docker

-------------------------------------------------------------------------------
docker run  -  Запуск контейнера
-------------------------------------------------------------------------------

Сигнатура
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG…]

Флаги:
-d      запуск контейнера в качестве отдельного процесса
-p      публикация порта

example:
-p 3000:3000      публикация открытого порта в интерфейсе хоста (HOST:CONTAINER)

-t      выделение псевдотерминала ?
-i      оставить STDIN открытым без присоединения к терминалу
--name  название контейнера
--rm    очистка системы при остановке/удалении контейнера
--restart   политика перезапуска
            no (default)
            on-failure[:max-retries] | always | unless-stopped
-e      установка переменной среды окружения
-e prodaction
-v      привязка распределенной файловой системы (name:/path/to/file)
-v mydb:/etc/mydb
-w      установка рабочей директории
\       разделение команд на строки

-------------------------------------------------------------------------------
Запускаем postgres контейнер
-------------------------------------------------------------------------------

docker run –rm \
#название контейнера
--name postgres \
#пользователь
-e POSTGRES_USER=postgres \
#пароль
-e POSTGRES_PASSWORD=12345 \
#название БД
-e POSTGRES_DB=mydb \
#автономный режим и порт
-dp 5432:5432 \
#том для хранения данных
-v $HOME/docker/volumes/postgres:/var/lib/postgresql/data \
#образ
postgres

-------------------------------------------------------------------------------
docker build
-------------------------------------------------------------------------------

docker build    создание образа на основе файла Dockerfile и контекста
docker build [OPTIONS] PATH | URL

.dockerignore   исключение файлов из сборки образа

-------------------------------------------------------------------------------
docker exec
-------------------------------------------------------------------------------

docker exec     выполнение команды в запущенном контейнере
docker exec [OPTIONS] CONTAINER COMMAND [ARGS…]

-d      выполнение команды в фоновом режиме
-e      установка переменной среды окружения
-i      оставить STDIN открытым
-t      выделение псевдотерминала
-w      определение рабочей директории внутри терминала
# -U пользователь, по умолчанию root

docker exec –it postgres psql –U postgres

-------------------------------------------------------------------------------
docker ps
-------------------------------------------------------------------------------

docker ps       получение списка контейнеров
-a              показать все контейнеры
-f              фильтрация вывода (id, name, status)
-n              показать n последних контейнеров
-l              показать последний созданный контейнер

doscker ps –f status=paused

-------------------------------------------------------------------------------
Контейнеры и образы
-------------------------------------------------------------------------------

docker images               получение списка образов
docker start CONTAINER      запуск контейнера
docker pause CONTAINER      пауза всех процессов контейнера
docker stop CONTAINER       остановка контейнера
docker kill CONTAINER       убийство контейнера
docker restart CONTAINER    перезапуск контейнера

docker rm [OPTIONS] CONTAINER
-f      принудительное удаление запущенного контейнера
-v      удаление анонимных томов

docker rmi IMAGE            удаление образа

docker image COMMAND        управление образами
docker container COMMAND    управление контейнерами
docker volume COMMAND       управление томами
docker network COMMAND      управление сетями

docker network inspect bridge   список сетей bridge
docker network ls               получаем список сетей
docker attach <name>            подключение к контейнеру

-------------------------------------------------------------------------------
docker system / logs
-------------------------------------------------------------------------------

docker system COMMAND       управление докером

docker logs [OPTIONS] CONTAINER
-f      следование за выводом
-n      n последних строк

docker system prune         БЕЗВОЗВРАТНОЕ удаление всех неиспользуемых контейнеров
-a                          удаление всех неиспользуемых контейнеров
--volumes                   удаление томов

===============================================================================
Dockerfile
===============================================================================

Dockerfile – документ без расширения, содержащий инструкции для создания образа
при выполнении docker build

!!! не использовать / в path !!!
Каждая инструкция выполняется независимо от других

-------------------------------------------------------------------------------
Инструкции Dockerfile
-------------------------------------------------------------------------------

FROM
FROM <image>[:<tag>] [AS <name>]
FROM node:12-alpine AS build        родительский образ

WORKDIR
WORKDIR /app                       установка рабочей директории

COPY
COPY <src> <dest>
COPY package.* ./                  копирование новых файлов

ADD                                добавить файлы

RUN <command>
RUN ["executable", "arg1", "arg2"]
RUN npm install                    выполнение команды в новом слое

CMD
CMD ["executable", "arg1", "arg2"]
CMD ["node", "app/src/index.js"]
! Выполняется только последняя CMD

ENTRYPOINT
ENTRYPOINT ["executable", "arg1", "arg2"]
ENTRYPOINT ["top", "-b"]

Переменные:
${var} / $var

EXMP
ENV FOO=/bar
WORKDIR ${FOO}

LABEL
LABEL <key>=<value>
LABEL version="1.0"

EXPOSE
EXPOSE <port> | <port>/<protocol>
EXPOSE 3000

ENV
ENV <key>=<value>
ENV MY_ENV="ENV"

VOLUME
VOLUME /var/log
docker volume prune

USER
USER <user>[:<group>]
USER <UID>[:<GID>]

ARG
ARG <name>[=<default value>]
docker build --build-arg <name>=<value>

ONBUILD
ONBUILD <INSTRUCTION>

.dockerignore

-------------------------------------------------------------------------------
Пример Dockerfile для Node приложения
-------------------------------------------------------------------------------

FROM node:16
WORKDIR /usr/src/app
COPY package*.json ./
RUN nmp install
COPY . .
EXPOSE 4000
CMD ["node", "server.js"]

===============================================================================
Docker-compose.yml
===============================================================================

Docker-compose.yml – инструмент для запуска приложений из нескольких контейнеров

docker-compose [-f <arg>…] [--profile <name>…] [options] [COMMAND] [ARGS…]

-f              путь к Docker-compose.yml
-p              название проекта
--project-path  альтернативная рабочая директория

up
down
start
stop
restart
create
rm
run
exec

-------------------------------------------------------------------------------
ФАЙЛ!!!
-------------------------------------------------------------------------------

services:
  webapp:
    build: ./dir

или объект

services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
        gitcommithash: c31dg

ARG buildno
ARG gitcommithash
RUN echo "Номер сборки": $buildno
RUN echo "Коммит собран": $gitcommithash

-------------------------------------------------------------------------------
#network / depends_on / restart_policy / env_file / ports / networks
(СОХРАНЕНО БЕЗ ИЗМЕНЕНИЙ — см. выше в тексте)
-------------------------------------------------------------------------------

===============================================================================
docker compose commands
===============================================================================

docker compose up -d
docker compose stop
docker compose rm

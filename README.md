================================================================================
DOCKER / DOCKERFILE / DOCKER COMPOSE — CHEATSHEET
================================================================================
DOCKER
================================================================================

docker run  -  Запуск контейнера
--------------------------------------------------------------------------------

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


Запускаем postgres контейнер
--------------------------------------------------------------------------------

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


docker build
--------------------------------------------------------------------------------

docker build        создание образа на основе файла Dockerfile и контекста
docker build [OPTIONS] PATH | URL

.dockerignore       исключение файлов из сборки образа


docker exec
--------------------------------------------------------------------------------

docker exec         выполнение команды в запущенном контейнере
docker exec [OPTIONS] CONTAINER COMMAND [ARGS…]

-d      выполнение команды в фоновом режиме
-e      установка переменной среды окружения
-i      оставить STDIN открытым
-t      выделение псевдотерминала
-w      определение рабочей директории внутри терминала
# -U пользователь, по умолчанию root

docker exec –it postgres psql –U postgres


docker ps
--------------------------------------------------------------------------------

docker ps           получение списка контейнеров
-a                  показать все контейнеры
-f                  фильтрация вывода (id, name, status)
-n                  показать n последних контейнеров
-l                  показать последний созданный контейнер

doscker ps –f status=paused


Контейнеры и образы
--------------------------------------------------------------------------------

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


docker system / logs
--------------------------------------------------------------------------------

docker system COMMAND       управление докером

docker logs [OPTIONS] CONTAINER
-f      следование за выводом
-n      n последних строк

docker system prune         БЕЗВОЗВРАТНОЕ удаление всех неиспользуемых контейнеров
-a                          удаление всех неиспользуемых контейнеров
--volumes                   удаление томов


================================================================================
DOCKERFILE
================================================================================

Dockerfile – документ без расширения, содержащий инструкции для создания образа
при выполнении docker build

!!! не использовать / в path !!!
Каждая инструкция выполняется независимо от других


Инструкции
--------------------------------------------------------------------------------

FROM
FROM <image>[:<tag>] [AS <name>]
FROM node:12-alpine AS build        родительский образ

WORKDIR
WORKDIR /app                       установка рабочей директории

COPY
COPY <src> <dest>
COPY package.* ./                  копирование новых файлов
                                   (можно копировать удаленные и автоматически
                                    разархивировать)

ADD                                добавить файлы

RUN <command>
RUN ["executable", "arg1", "arg2"]
RUN npm install                    выполнение команды в новом слое и фиксация

CMD
CMD ["executable", "arg1", "arg2"]
CMD ["node", "app/src/index.js"]   дефолтные значения
! Выполняется только последняя CMD

ENTRYPOINT
ENTRYPOINT ["executable", "arg1", "arg2"]
ENTRYPOINT ["top", "-b"]           настройка исполняемого контейнера

Переменные:
${var} / $var

EXMP
ENV FOO=/bar
WORKDIR ${FOO}

LABEL
LABEL <key>=<value>
LABEL version="1.0"                добавление метаданных

EXPOSE
EXPOSE <port> | <port>/<protocol>
EXPOSE 3000                        сетевой порт

ENV
ENV <key>=<value>
ENV MY_ENV="ENV"                   переменная окружения

VOLUME
VOLUME /var/log                    ??? создание точки монтирования
docker volume prune                удаление неиспользуемых томов

USER
USER <user>[:<group>]
USER <UID>[:<GID>]                 установка пользователя

ARG
ARG <name>[=<default value>]
docker build --build-arg <name>=<value>

ONBUILD
ONBUILD <INSTRUCTION>              добавление триггера

.dockerignore                      исключение файлов из сборки


Пример для node приложения
--------------------------------------------------------------------------------

FROM node:16
WORKDIR /usr/src/app

COPY package*.json ./
RUN nmp install

COPY . .

EXPOSE 4000
CMD ["node", "server.js"]

установка зависимостей, символ *
копируем исходный код


================================================================================
DOCKER COMPOSE
================================================================================

Docker-compose.yml – инструмент для запуска docker-приложений,
состоящих из нескольких контейнеров

docker-compose [-f <arg>…] [--profile <name>…] [options] [COMMAND] [ARGS…]

-f              путь к Docker-compose.yml
-p              название проекта
--project-path  альтернативная рабочая директория

up              создание и запуск сервисов
down            остановка и удаление контейнеров, сетей, образов
start           запуск сервисов
stop            остановка сервисов
restart         перезапуск
create          создание
rm              удаление остановленных контейнеров
run             выполнение одноразовой команды
exec            выполнение команды в запущенном контейнере


ФАЙЛ!!!
--------------------------------------------------------------------------------

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


заданные аргументы можно использовать в докерфайлах
--------------------------------------------------------------------------------

ARG buildno
ARG gitcommithash
RUN echo "Номер сборки": $buildno
RUN echo "Коммит собран": $gitcommithash


#network
--------------------------------------------------------------------------------

build:
  context: .
  network: host

build:
  context: .
  network: custom_network_1        сеть доступна во время RUN


#depends_on
--------------------------------------------------------------------------------

services:
  web:
    build: .
    depends_on:
      - db
      - redis

redis:
  image: redis

db:
  image: postgres                  сервисы запускаются в зависимости


#restart_policy
--------------------------------------------------------------------------------

services:
  redis:
    image:
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max-attempts: 3
        window: 120s

none, on-failure, any

restart: "no"
restart: always
restart: on-failure
restart: unless-stopped

-в любом случае
-в случае аварийной остановки
-всегда кроме преднамеренной остановки

entrypoint: /code/entrypoint.sh    перезапись дефолтной точки входа


#env_file
--------------------------------------------------------------------------------

env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/runtime_opts.env


ports / expose
--------------------------------------------------------------------------------

expose:
  - "3000"
  - "8000"                         без публикации на хосте

external_links:
  - redis_1
  - project_db_1:mysql1
  - project_db_1:postgresql


network_mode
--------------------------------------------------------------------------------

"bridge"
"host"
"none"


#networks
--------------------------------------------------------------------------------

services:
  some-service:
    networks:
      - some-netwok
      - other-network


ports
--------------------------------------------------------------------------------

"3000"
"8000:8000"
"9090-9091:8080-8081"
"127.0.0.1:8001:8001"
"127.0.0.1:5000"
"6060:6060/udp"

длинный синтаксис:
target: порт контейнера
published: порт хоста
protocol: tcp|udp
mode: host | ingress

(IPADDR:HOSTPORT:CONTAINERPORT)


Замена переменных
--------------------------------------------------------------------------------

db:
  image: "postgres:${POSTGRES_VERSION}"

docker compose up POSTGRES_VERSION=9.3

${VAR:-default}
${VAR?error}

Compose использует переменные из терминала или .env
Переменные из терминала перезаписывают .env

дефолтное значение
ошибка если переменная отсутствует


================================================================================
ПРИМЕР КОНТЕЙНЕРИЗАЦИИ CLIENT / ADMIN / SERVER
================================================================================

# client dockerfile
ARG NODE_VERSION=16.13.1
FROM node:$NODE_VERSION
WORKDIR /client
COPY package*.json
RUN npm install
COPY . .
RUN npm run build

# admin dockerfile
ARG NODE_VERSION=16.13.1
FROM node:$NODE_VERSION as build
WORKDIR /admin
COPY package*.json
RUN npm install
COPY . .
RUN yarn build

# api
ARG NODE_VERSION=16.13.1
FROM node:$NODE_VERSION
COPY package*.json
RUN npm install
COPY . .
EXPOSE 5000
CMD ["nmp", "start"]


.env (в корне проекта)
--------------------------------------------------------------------------------

APP_NAME=my-app
NODE_VERSION=16.13.1
POSTGRES_VERSION=14
POSTGRES_USER=postgres
POSTGRES_PASSWORD=12345
POSTGRES_DB=mydb

DATABASE_URL=postgresql://postgres"postgres@12345:5432/mydb?schems=public
# ?? почему public ???


# .dockerignore (в каждом сервисе)
--------------------------------------------------------------------------------

node_modules
npm.error.log
и тд


version: "3.9"
--------------------------------------------------------------------------------

services:
  postgres:
    env_file: .env
    container_name: ${APP_NAME}_postgres
    image: postgres:${POSTGRES_VERSION}
    volumes:
      - data_postgres:/var/lib/postgresql/data
    ports:
      - 5432:5432
    networks:
      - app-network
    restart: on-failure

  client:
    env_file: .env
    container_name: ${APP_NAME}_client
    image: node:${NODE_VERSION}
    working_dir: /app
    volumes:
      - ./services/client:/app:rw
    depends_on:
      - api
    ports:
      - 3000:3000
    networks:
      - app-network
    restart: on-failure
    command: bash –c "npm run start"

  admin:
    env_file: .env
    container_name: ${APP_NAME}_admin
    image: node:${NODE_VERSION}
    working_dir: /app
    volumes:
      - ./services/admin:/app:rw
    depends_on:
      - api
    ports:
      - 4000:4000
    networks:
      - app-network
    restart: on-failure
    command: bash –c "npm run start"

  api:
    env_file: .env
    container_name: ${APP_NAME}_admin
    build: services/api
    ports:
      - 5000:5000
    depends_on:
      - postgres
    restart: on-failure
    command: bash –c "npm run dev"

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx
      - ./certbot/www
      - ./certbot/conf
    ports:
      - "80:80"
      - "443:443"

volumes:
  data_postgres:


docker compose commands
--------------------------------------------------------------------------------

docker compose up -d
docker compose stop
docker compose rm

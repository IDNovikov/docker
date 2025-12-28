# 🐳 Docker / Dockerfile / Docker Compose — Шпаргалка

===============================================================================
## 1. DOCKER — ОСНОВНЫЕ КОМАНДЫ
===============================================================================

### docker run — запуск контейнера

Сигнатура:
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG…]

Флаги:
-d	запуск контейнера в качестве отдельного процесса (detached)
-p	публикация порта  
example:
-p 3000:3000	публикация открытого порта в интерфейсе хоста (HOST:CONTAINER)

-t	выделение псевдотерминала (TTY)
-i	оставить STDIN открытым без присоединения к терминалу
--name	название контейнера
--rm	очистка системы при остановке/удалении контейнера
--restart	политика перезапуска  
	no (default)  
	on-failure[:max-retries]  
	always  
	unless-stopped
-e	установка переменной среды окружения  
-e production
-v	привязка распределенной файловой системы (name:/path/to/file)  
-v mydb:/etc/mydb
-w	установка рабочей директории
\	разделение команд на строки

-------------------------------------------------------------------------------

### Пример: запуск postgres контейнера

docker run --rm \
# название контейнера
--name postgres \
# пользователь
-e POSTGRES_USER=postgres \
# пароль
-e POSTGRES_PASSWORD=12345 \
# название БД
-e POSTGRES_DB=mydb \
# автономный режим и порт
-dp 5432:5432 \
# том для хранения данных
-v $HOME/docker/volumes/postgres:/var/lib/postgresql/data \
# образ
postgres

===============================================================================
## 2. DOCKER BUILD / EXEC / PS
===============================================================================

### docker build — сборка образа

docker build [OPTIONS] PATH | URL

.dockerignore — исключение файлов из сборки образа

-------------------------------------------------------------------------------

### docker exec — выполнение команды в контейнере

docker exec [OPTIONS] CONTAINER COMMAND [ARGS…]

Флаги:
-d	выполнение команды в фоновом режиме
-e	установка переменной среды окружения
-i	оставить STDIN открытым
-t	выделение псевдотерминала
-w	рабочая директория
# -U пользователь, по умолчанию root

Пример:
docker exec -it postgres psql -U postgres

-------------------------------------------------------------------------------

### docker ps — контейнеры

docker ps	список контейнеров
-a	все контейнеры
-f	фильтрация (id, name, status)
-n	n последних контейнеров
-l	последний созданный контейнер

docker ps -f status=paused

===============================================================================
## 3. УПРАВЛЕНИЕ КОНТЕЙНЕРАМИ И ОБРАЗАМИ
===============================================================================

docker images	список образов

docker start CONTAINER
docker pause CONTAINER
docker stop CONTAINER
docker kill CONTAINER
docker restart CONTAINER

docker rm CONTAINER
-f	принудительное удаление
-v	удаление анонимных томов

docker rmi IMAGE

docker image COMMAND
docker container COMMAND
docker volume COMMAND
docker network COMMAND

-------------------------------------------------------------------------------

### Сети

docker network ls
docker network inspect bridge

docker attach <name>	подключение к контейнеру

===============================================================================
## 4. ЛОГИ И ОЧИСТКА
===============================================================================

docker logs [OPTIONS] CONTAINER
-f	следование за выводом
-n	n последних строк

docker system COMMAND

docker system prune	БЕЗВОЗВРАТНОЕ удаление
-a	все неиспользуемые контейнеры и образы
--volumes	удаление томов

===============================================================================
## 5. DOCKERFILE
===============================================================================

Dockerfile — файл без расширения  
Каждая инструкция выполняется независимо (новый слой)  
Выполняется только последняя CMD

### Инструкции

FROM  
FROM <image>[:<tag>] [AS <name>]  
FROM node:12-alpine AS build

WORKDIR  
WORKDIR /app

COPY  
COPY <src> <dest>  
COPY package.* ./

ADD	добавление файлов

RUN <command>  
RUN ["executable","arg1","arg2"]  
RUN npm install

CMD  
CMD ["executable","arg1","arg2"]  
CMD ["node","app/src/index.js"]

ENTRYPOINT  
ENTRYPOINT ["executable","arg1","arg2"]

Переменные:
${var} / $var

EXAMPLE:
ENV FOO=/bar  
WORKDIR ${FOO}

LABEL  
LABEL version="1.0"

EXPOSE  
EXPOSE 3000

ENV  
ENV MY_ENV="ENV"

VOLUME  
VOLUME /var/log  
docker volume prune

USER  
USER <user>[:<group>]

ARG  
ARG <name>=<default>  
docker build --build-arg name=value

ONBUILD  
ONBUILD <INSTRUCTION>

===============================================================================
## 6. DOCKERFILE — ПРИМЕР NODE.JS
===============================================================================

FROM node:16
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 4000
CMD ["node","server.js"]

===============================================================================
## 7. DOCKER COMPOSE
===============================================================================

Docker-compose.yml — запуск многоконтейнерных приложений

docker-compose [OPTIONS] COMMAND

-f	путь к файлу
-p	название проекта

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

### Пример файла

services:
  webapp:
    build: ./dir

или

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
RUN echo "Номер сборки: $buildno"
RUN echo "Коммит: $gitcommithash"

===============================================================================
## 8. NETWORK / DEPENDS / RESTART
===============================================================================

build:
  context: .
  network: host

build:
  context: .
  network: custom_network_1

depends_on:
  - db
  - redis

restart:
  no
  always
  on-failure
  unless-stopped

deploy:
  restart_policy:
    condition: on-failure
    delay: 5s
    max-attempts: 3
    window: 120s

===============================================================================
## 9. ENV / PORTS / NETWORK_MODE
===============================================================================

env_file:
  - .env
  - ./common.env
  - ./apps/web.env

expose:
  - "3000"
  - "8000"

ports:
  - "3000"
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "127.0.0.1:8001:8001"
  - "6060:6060/udp"

network_mode:
  bridge
  host
  none

===============================================================================
## 10. ПЕРЕМЕННЫЕ И ПРИМЕР КОНТЕЙНЕРИЗАЦИИ
===============================================================================

${VAR:-default}
${VAR?error}

CLIENT / ADMIN / API — см. Dockerfile выше

.env:
APP_NAME=my-app
NODE_VERSION=16.13.1
POSTGRES_VERSION=14
POSTGRES_USER=postgres
POSTGRES_PASSWORD=12345
POSTGRES_DB=mydb

DATABASE_URL=postgresql://postgres:12345@postgres:5432/mydb?schema=public

===============================================================================
## 11. КОМАНДЫ
===============================================================================

docker compose up -d
docker compose stop
docker compose rm

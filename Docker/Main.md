Основы Docker.

В Docker есть 2 основных понятия - это docker-image и docker-container. Docker-image - это это сборка, готовое к запуску приложение, но ещё не запущенное. Docker-container - это работающее приложение, созданное на базе docker-image. На основе одного и того же образа можно создать кучу одинаковых контейнеров.

Для контейнера образ является read-only системой, которую он никаким образом не может изменить.

Image представляет собой "слоёный пирог". Например, есть образ ubuntu. Никакой полезной работы с его помощью сделать нельзя, но просто запустить контейнер на основе этого образа мы всё же можем. Но задача стоит запустить nginx. Для этого мы собираем образ на основе образа ubuntu. Это уже новый образ! На основе этого нового образа можно собирать другие образы!

Чтобы посмотреть, какие образы уже есть в локальном registry, выполним команду:

$ docker images

Чтобы увидеть, какие контейнеры сейчас запущены, используем команду:

$ docker ps

Для того, чтобы собрать docker-образ, используется комманда:

$ docker build -t <image_name> .

После всех параметров нужно указать путь к директории, в которой находится Dockerfile. Если это текущая директория, то указываем точку (.)

Синтаксис Dockerfile.

FROM - это базовый образ, с которого мы начинаем сборку. Синтаксис: FROM <name>:<tag>. Например, FROM python:3.6
RUN - определяет, что нужно выполнить указанную команду. Например, RUN mkdir -p /usr/src/app
WORKDIR - переход в указанный каталог. Например, WORKDIR /usr/src/app
COPY - команда копирования. Синтаксис: COPY <from> <to>. Например, COPY . /usr/src/app. <from> - это путь на нашей машине. <to> - путь в контейнере.
CMD - команда, которая говорит, что нужно делать, когда мы запустим контейнер. Например, CMD ["python", "app.py"]. Запуск осуществляется через shell.
ENTRYPOINT - команда, аналогичная CMD. Но она выполняется без shell.
EXPOSE - указание контейнеру пробросить наружу порт. Например, EXPOSE 8080. Но это просто декларация, порт на самом деле проброшен не будет. Чтобы порт был на самом деле проброшен, нужно указать это при запуске контейнера:

$ docker run --name <container_name> -p <local_port>:<container_port> <image_name>

Каждая команда, указанная в Dockerfile, создаёт НОВЫЙ СЛОЙ! Когда docker делает pull из registry, можно наблюдать выполнение всех слоёв, из которых состоит образ.

Для запуска образа используется команда:

$ docker run <image_name>

Будет запущен контейнер на основе указанного образа. Важно понимать, что контейнер будет запущен до тех пор, пока работает приложение!

Чтобы посмотреть все контейнеры, включая остановленные, используется команда:

$ docker ps -a

Можно попросить Docker выводить только ID'шники контейнеров, с помощью параметра -q:

$ docker ps -a -q

Если при запуске контейнера не указать имя, то Docker сам установить имя контейнера. Для задания своего имени контейнеру используется ключ --name:

$ docker run --name <container_name> <image_name>

Для удаления контейнера используется команда:

$ docker rm <container_id | container_name>

Для удаления всех контейнеров используется композиция команд:

$ docker rm $(docker ps -aq)

Чтобы запустить контейнер в фоне используется ключ -d:

$ docker run -d --name <container_name> <image_name>

Для остановки контейнера используется команда:

$ docker stop <container_id | container_name>

Для запуска уже существующего контейнера используется команда:

$ docker start <container_id | container_name>

Можно запустить контейнер таким образом, что после завершения работы (самостоятельно или принудительно) он автоматически удалился:

$ docker run --name <container_name> -d --rm <image_name>

В контейнер можно пробрасывать переменные окружения несколькими способами. Первый - это указать переменную окружения прямо в Dockerfile с помощью директивы ENV:

ENV <env_name> <env_value>

Минус этот способа в том, что при изменении переменных окружения нужно пересобирать образ.

Другой способ состоит в том, чтобы указать переменную окружения при запуске контейнера через ключ -e:

$ docker run -e <env_name>=<env_value> <image_name>

Важно помнить, что эта переменная окружения запоминается на уровне контейнера. Т.е. если остановить контейнер командой stop, а затем запустить его командой start, то для переменной будет взято то значение, которое было указано при команде run.

К контейнеру можно примонтировать папку. Делается при создании и запуске контейнера командой run указанием параметра -v:

$ docker run -v <local_folder_absolute_path>:<container_folder_absolute_path> <image_name>

Приложение может изменять файлы в примонтированной папке из контейнера!

Есть другой способ монтирования папки к контейнеру - через Docker Volume! Docker Volume представляет собой папку, которая хранится по определённому пути.

Чтобы посмотреть список доступных volume, используется команда:

$ docker volume ls

Для создания volume используется команда:

$ docker volume create <volume_name>

После этого создастся пустой volume. Теперь можно немного модифицировать команду для запуска контейнера:

$ docker run -v <volume_name>:<container_folder_absolute_path> <image_name>

Теперь все файлы по container_folder_absolute_path хранятся в указанном volume. Это актуально для баз данных или для файловых серверов. После остановки или удаления контейнера с данными в volume ничего не происходит. Этот же volume можно привязать к другому контейнеру.

Чтобы сделать attach к уже запущенному контейнеру, нужно выполнить команду:

$ docker attach <container_name>

Если контейнер был запущен с ключами -it, то из открывшейся сессии shell можно выйти комбинацией <Ctrl-p><Ctrl-q>. Если же ключи -it указаны не были, то выйти из shell можно будет только при завершении работы контейнера комбинацией <Ctrl-c>!

Команда attach позволяет прицепиться к процессу, указанному в docker-файле через директиву CMD. Если нужно просто открыть shell в контейнере, то можно использовать следующую команду:

$ docker exec -it <container_name> /bin/bash

Для данной команды не нужно запускать контейнер с ключами -it, это ни на что не повлияет. Из запущенного shell можно выйти через exit, но контейнер продолжит свою работу.
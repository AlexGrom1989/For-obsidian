```powershell
git clone https://gitlab.bmstu.ru/devops-dataops-intro/labs/2-docker.git
 
# Set up Docker's apt repository.
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# To install the latest version, run:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify that the Docker Engine installation is successful by running the hello-world image.
sudo docker run hello-world

# Добавляем группу docker (-aG docker) и пользователя alex ($USER), чтобы  пользователь alex мог использовать docker без sudo
sudo usermod -aG docker $USER

# -i (interective) интерактивное взаимодействие через консоль, -t выделение виртуального терминала, --name mycontainer название нашего контейнера, ubuntu - название образа из которого будет создан контейнер, bash - запись в параметр CMD нашему образу
sudo docker run -it --name mycontainer ubuntu bash

# Выйти из контейнера. Это можно написать в терминале установленного ubuntu в нашем контейнере
exit 

# Войдем обратно в терминал нашей новой ubuntu
# Сначала посмотрим на запущенные образы
sudo docker ps -a
sudo docker start mycontainer
sudo docker attach mycontainer

# Удаление контейнера
sudo docker rm -f mycontainer

# Установка ps в терминале новой ubuntu со всеми дополнениями
apt update
apt install psmisc

# Docker статистика. пишем в старом трминале
docker stats

# Если bash контейнера занят вычислениями, то мы можем не отвлекая его создать новый вход в этот bash
docker exec -it mycontainer bash

# Список запущенных bash
pstree -p


### Теперь создадим проект и напишем инструкции к нашему докер контейнеру

# Создаем файл Dockerfile, это будет файл с командами запуска контейнера.
# В нем пишем:
FROM python:3.12

COPY requirements.txt /requirements.txt
RUN  pip install -r /requirements.txt && \
     rm /requirements.txt

WORKDIR /app
COPY src/*.py .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]

# Подготовка образа
# В cmd проекта пишем для построения контейнера по нашим инструкциям
sudo docker build . -t mymyimage:1.0.0

# Переименование созданного образа
sudo docker tag mymyimage:1.0.0 myimage:1.0.0

# Создать контейнер на основе образа
sudo docker run -it --name mycontainer myimage:1.0.0

# Перейти в командную строку нашего контейнера
sudo docker exec -it mycontainer bash

# Удаление ненужного старого контейнера 
sudo docker rm mycontainer

# Создание контейнера из образа связав его с портом 8000:8000
sudo docker run -it --rm --name mycontainer -p 8000:8000 myimage:1.0.0

# или так, но с подключением базы данных
sudo docker run -it --rm --name mycontainer -p 8080:8000 -v $PWD/src/test.db:/app/test.db myimage:1.0.0

### И вауля перехдоим по адресу http://127.0.0.1:8080/docs 

/* ========================================================== */
/*                    Конспекты от препода                    */
/* ========================================================== */

/* -------------------------- */
/* Часть 1 | Установка Docker */
/* -------------------------- */
Docker является наиболее распространённым средством упарвления контейнерами.
Современные версии Docker реализацуют концепцию разделения API CNFC и зависит от containerd и runc.
Для начала установки в ОС Ubuntu следует выполнить следующие команды:

# Добавление официальных GPG ключей
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Добавление репозитория Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update


Собственно установка Docker и всех зависимостей:

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


Проверка корректности работы Docker выполняется запуском простого контейнера:

sudo docker run hello-world


Для того чтобы получить возможность использовать утилиту Docker без указания sudo следует выполнить следующие операции:

sudo usermod -aG docker $USER

sudo systemctl restart docker

su - $USER


Однако, при закрытии терминала, до выхода из сессии пользователя ОС следует повторять последнюю строчку каждый раз при запуске нового терминала.


/* --------------------------------- */
/* Часть 2 | Основные команды Docker */
/* --------------------------------- */
Для того чтобы проверить корректность работы Docker Client и Docker Deamon следует выполнить команду:

docker version


Простой запуск контейнера, который выведет информацию и вернёт управление хост ОС:

docker run hello-world


Вывод результата выполнения контейнера будет иметь вид:

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:d211f485f2dd1dee407a80973c8f129f00d54604d2c90732e8e320e5038a0348
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/


Однако, первые 5 строчек информации являются сервисными сообщениями Docker о невозможности найти образ hello-world:latest в локальном хранилище и логами процесса загрузки образа из удалённого хранилища.
Далее, информационное сообщение содержит описание действий, выполненных Docker для вывода показанного сообщения.
Более комплексный пример управления контейнером является запуск контейнера ОС Ubuntu:

docker run -it ubuntu bash


Кооманда запуска включает в себя:


Флаги -i и -t, позволяющие использовать интерактивный режим работы и создать виртуальный tty для передачи команд управления ОС контейнера


Название образа контейнера ubuntu


Название выполняемой при старте команды bash


Следует заметит, что после запуска контейнера обозначение текущего пользователя и имени хоста будут изменены на root@[container_id], т.е. на суперпользователя ОС контейнера и идентификатор контейнера.
Выполним команду получения сведений об ОС:

cat /etc/os-release


Команда получения текущих процессов ps покажет существование только двух процессов:

    PID TTY          TIME CMD
      1 pts/0    00:00:00 bash
      9 pts/0    00:00:00 ps


Установим и используем утилиту pstree (не входит в стандартную поставку контейнера Ubuntu):

apt update
apt-get install psmisc

pstree


Результат выполнения операции будет содержать информацию о наличии всего одного процесса в ОС:

bash---pstree


Поскольку контейнер не нуждается в инциализации пользовательских процессов и комплексных средств многозадачности и управления ресурсами (таких как systemd), то при запуске контейнер может сразу осуществлять целевое назначение.
Выполним эмуляцию целевой нагрузки контейнера выполнив скрипт:

for i in {001..100}; do sleep $i; echo $i; done


В новой консоли выполним проверку запущенных контейнеров:

docker ps


Вывод будет содержать основные сведения о контейнере:

CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
655e865433e8   ubuntu    "bash"    7 minutes ago   Up 7 minutes             busy_cohen


Стоит заметить, что поскольку мы не указывали имени контейнера, оно было сгенерировано автоматически.
Указать имя контейнера можно при помощи флага --name [container_name] при запуске контейнера.
Можно также отобразить уже остановленные контейнеры:

docker ps -a


Остановленные контейнеры можно удалить при помощи команды:

docker rm "[container_id|container_name]"


Также для автоматического удаления контейнера после остановки (окончания начального процесса) для команды run можно добавить флаг --rm.
Для того чтобы подключиться к контейнеру при помощи дополнительного виртуального терминала следует выполнить команду:

docker exec -it "[container_id|container_name]" bash


Такое подключение не затронет уже запущенный процесс поскольку будет запущенно в отдельном виртуальном терминале, при этом pstree не сможет отобразить процессы между отдельными виртаульными терминалами.
При этом, второму виртуальному терминалу будет доступна вся ОС.
Таким образом, например, удобно осуществлять запуск сервисных сценариев космплексных приложений в контейнерах.
Посмотреть текущую нагрузку контейнеров на хост систему можно при помощи команды:

docker stats


Посмотреть процессы контейнера можно при помощи команды:

docker top


Остановим (без удаления) текущий контейнер. Для этого следует завершить все процессы в контейнере или выполнить команду:

docker restart "[container_id|container_name]"


Однако, контейнер автоматически удалится если при запуске был указан флаг --rm.
Для повторного его запуска можно использовать команды:

docker restart "[container_id|container_name]"
# или
docker start "[container_id|container_name]"


И затем повторно подключиться к виртуальному терминалу контейнера:

docker attach "[container_id|container_name]"

/* ---------------------------------- */
/*  Часть 3 | Докеризация приложения  */
/* ---------------------------------- */

Выполним "контейнеризацию" приложения. Т.е. сконструируем для зараннего подготовленного приложения образ контейнера и запустим его.

Задание 1 | Запуск приложения
В отчёте следует указать описание использованного приложения
Выполним установку зависимостей и локальный запуск приложения:

# Установка зависимостей
pip install -r requirements.txt

# Запуск приложения
cd src
uvicorn main:app


При переходе по адресу http://127.0.0.1:8000/docs/ будет доступно описание API приложения с возможностью передать в него данные.
Приложение сохраняет данные в sqlite3 БД src/test.db.
Примеры запросов, позволяющие получить результаты в консоли:

# Запрос всех заданий
curl --silent http://127.0.0.1:8000/tasks/ | jq

# Создание нового задания
curl --silent -X POST http://127.0.0.1:8000/tasks/ --data '{ "title": "Прочитать книгу", "description": "Закончить чтение книги по FastAPI", "completed": fals
e }' | jq



Задание 2 | Докеризация приложения
В отчёте следует указать выявленные проблемы взаимодействия с контейнером и пути их решения
Для контейнеризации приложения следует прежде всего создать Dockerfile. Этот файл будет содержать описание подготовки образа контейнера. Файл следует создать в корне проекта.

# Используем в качестве основы образ, предназначенный для работы с python
FROM python:3.12

# Установка зависимостей
COPY requirements.txt /requirements.txt
RUN  pip install -r /requirements.txt && \
     rm /requirements.txt

# Создаём рабочую директорию и копируем туда исполняемые файлы
WORKDIR /app
COPY src/*.py .

# Команда, используемая для запуска основного процесса
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]


Стоит обратить внимание, для команды запуска процесса передан флаг --host 0.0.0.0, который позволяет обращаться к приложению с использованием любого ip адреса, не только 127.0.0.1.
Подготовка образа выполняется следующей командой:

docker build . -t myimage


Где myimage - это название образа контейнера (тег образа), а . указывает на директорию относительно которой будет выполнена сборка - текущая директория.
Хорошей практикой является уточнение тега: добавление версии приложения через двоеточие. Например, myimage:1.0.0, это позволит избавиться от дублирования имёт образов контейнеров, что полезно для быстрого изменения версий.
Для запуска образа следует использовать команду:

docker run -it --name mycontainer --rm myimage


Следует обратить внимание, что к контейнеру на данный момент невозможно подключиться с хост ОС, при этом в нём не сохранены предыдущие данные
Однако, если выполнит команду:

docker exec -it mycontainer 


Для публикации образа в сервисе dockerhub следует использовать команду:

docker push mycontainer


Однако, это потребует авторизации в сервисе dockerhub и сздание собственного репозитория, что не будет рассмотрено в настоящей ЛР.

Задание 3 | Исправление запуска контейнера приложения
В отчёте следует описать модификации, которые потребовались для исправления запуска контейнера приложения
Для исправления запуска контейнера приложения следует выполнить команду:

docker run -it --rm -p 8080:8000 -v $(pwd)/src/test.db:/app/test.db {image_tag}


Здесь:


-p 8080:8000 позволит опубликовать порт приложения в хост ОС, при этом порт 8080 хост ОС будет соответствовать порту 8000 контейнера


$PWD/src/test.db:/app/test.db позволит связать файл БД хост ОС с файлом БД контейнера.


Особенности именования внутренних хранилищ Docker требуют указывать полный абсолютный путь для связывания файлов с Docker (для этого применена переменная окружения $PWD).
Стоит учитывать, что несуществующие в хост ОС файлы Docker-ом будут созданы как директории.

Часть 3 | Лучшие практики докеризации
Простая контейнеризация приложений на данный момент не отвечает требованиям удобства, скорости и безопасности развёртывания.
Поэтому следует добавить прослойку безопасности для старта приложения.
В качестве реверс-проксти предлагается использовать сервис nginx с самоподписанными сертификатами.

Задание 4 | Добавление реверс прокси
В отчёте следует описать процесс добавления реверс прокси
Для обеспечения безопасности, сгенерируем самоподписанные сертификаты при помощи утилиты openssl:

openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes


Описание старта двух контейнеров в рамках единого стека описано в файле docker-compose.yaml.
Файл содержит описание сервисов, которые являются контейнерами с заданными подключениями к файловой системе хост ОС, доступные порты и переменные окружения.
Также описаны секреты и конфигурации nginx.
Запуск осуществляется выполнением команды:

docker compose up


При этом, порты, по которым будет доступно приложение можно получить при помощи docker ps.
Например, если вывод команды представляет собой:

CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                                                                                    NAMES
5f92984a63e6   nginx        "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:32768->80/tcp, [::]:32768->80/tcp, 0.0.0.0:32769->443/tcp, [::]:32769->443/tcp   dev-ops-lab2-nginx-1
dfdee97704ef   test:1.0.0   "uvicorn main:app --…"   3 seconds ago    Up 2 seconds                                                                                             dev-ops-lab2-tasks-1


То по запросу https://localhost:32769/docs и после принятия предупреждения о самоподписанных сертификатах, будет доступна полностью рабочая версия приложения.

Задание 5 | Добавленеие версионнсти API | Факультативное задание
Факультативное задание, в отчёте можно описать решения для заданной программы конфигурирования пути доступа к приложению при помощи переменной окружения
Примерный код следующий:

import os

PATH_PREFIX = os.environ.gen("PATH_PREFIX", "v1")

@app.get(f"/{PATH_PREFIX}/tasks/", response_model=list[schemas.Task])
```
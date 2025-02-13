# Домашняя работа 19
-------------------------------------------------

## Тема: Docker: основы работы с контейнеризацией 

## Домашнее задание:

Цель домашнего задания: Разобраться с основами docker, с образом, эко системой docker в целом;

Задание:

1. Установите Docker на хост машину https://docs.docker.com/engine/install/ubuntu/

2. Установите Docker Compose - как плагин, или как отдельное приложение

3. Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)

4. Определите разницу между контейнером и образом. Вывод опишите в домашнем задании.

5. Ответьте на вопрос: Можно ли в контейнере собрать ядро?

6. Собранный образ необходимо запушить в docker hub и дать ссылку на ваш репозиторий.

Команды для выполнения ДЗ
<details>
 
- docker ps - просмотреть список запущенных контейнеров
- docker ps -a - просмотреть список всех контейнеров
- docker run -d -p port:port container_name - запуск нового контейнера с пробросом портов
- docker stop container_name - остановка контейнера
- docker logs container_name - вывод логов контейнеров
- docker inspect container_name - информация по запущенному контейнеру
- docker build -t dockerhub_login/reponame:ver - билд нового образа
- docker push/pull - отправка/получение образа из docker-registry
- docker exec -it container_name bash - выполнить команду внутри оболочки контейнера (в данном примере мы выполняем команду “bash” внутри контейнера и попадаем в оболочку, внутрь контейнера)

</details>

```
Для выполнения задания подготовлен Vagrantfile с предварительными настройками Docker Container.
```

#### Задание №1-2  Установите Docker и Docker Compose на хост машину

Настраиваем репозиторий Docker:
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
Установка компонентов Docker:

```sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin```

Создадим группу и добавим туда пользователя, чтобы работать не от sudo.

```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

Проверка успешной установки.
```sudo docker run hello-world```
<details>
<summary> Результат </summary>

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
</details>

Данный алгоритм установки взять с официального сайта Docker.

#### Задание №3 Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)

Создаём директорию, где будут расположены конфигурационные файлы образа докера
```mkdir -p /opt/nginxotus```

Создаём Dockerfile и index.html
```
echo 'FROM nginx:alpine
COPY ./index.html /usr/share/nginx/html/index.html' > /opt/nginxotus/Dockerfile
echo 'Hello world. Otus docker test' > /opt/nginxotus/index.html
```
Создаём образ Docker

```docker build -t nginxotus /opt/nginxotus -f /opt/nginxotus/Dockerfile```

Проверяем: ```docker ps -a```

<details>
 
<summary> Результат </summary>

```
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS                          PORTS                  NAMES
c0a7b886067e   hello-world   "/hello"                 About a minute ago   Exited (0) About a minute ago                          elastic_goldstine
3bd3c53e39d4   nginxotus     "/docker-entrypoint.…"   22 hours ago         Exited (255) 6 minutes ago      0.0.0.0:8080->80/tcp   webserver
```

</details>

Запускаем контейнет из образа

```docker run --name nginxotus -d -p 8080:80 nginxotus```

Проверяем:

<details>
 
<summary> docker ps -a </summary>

```
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                  NAMES
3bd3c53e39d4   nginxotus   "/docker-entrypoint.…"   12 seconds ago   Up 11 seconds   0.0.0.0:8080->80/tcp   nginxotus
```

<summary> curl 127.0.0.1:8080 </summary>

```
Hello world. Otus docker test
```

</details>

#### Задание №4 Определите разницу между контейнером и образом. Вывод опишите в домашнем задании.
Docker контейнер - это экземпляр запущенного Docker образа. Образ - это своего рода шаблон, а контейнер - это конкретная реализация этого шаблона, которая выполняется в определённый момент времени.

- Исполнаяемый.статичный
  - Образ - статичен и используется для хранения компонентов приложения
  - Контейнер - динамичен и является экземпляром запущенного образа
- Состояние
  - Образ - неизменяемый (immutable). После создания его нельзя изменить, но можно создать новый образ на основесуществующего
  - Контейнер - изменяемый. Внутри него могут происходить изменения состояния приложения, но эти изменения не сохраняются в образе
- Запуск
  - Образ сам по себе не может быть запущен. Для этого необходимо создать контейнер
  - Контейнер запускается командой и начинает выполнение кода, заложенного в образе
- Переносимость
  - Образ можно легко переносить между разными машина через репозитории, например Docker HUB
  - Контейнер привязан к конкретной машине и обычно не переноситя целиком, т.к. содержит временные данные.

#### Задание №5 Ответьте на вопрос: Можно ли в контейнере собрать ядро?

В Docker-контейнере можно собрать ядро. Для этого, контейнер необходимо запустить с расширенными правами (ключ: --privileged)

Это удобный способ изоляции среды разработки и тестирования, особенно если вам нужно собрать несколько версий ядра или протестировать различные версии конфигурации, однако этот процесс достаточно ресурсоемкий, требующий доступа к системным библиотекам компилятора.

#### Задание №6 Собранный образ необходимо запушить в docker hub и дать ссылку на ваш репозиторий.

Зарегистрируемся на Docker Hub.
Далее, выполним команду для подключения к Docker Hub:

```docker login```

Чтобы загрузить изображения в хаб, необходимо создать ТЭГ, используя локальное изображение:

```sudo docker tag nginxotus pahami/nginxotus:latest```

Отправляем образ в hub:

```docker push pahami/nginxotus:latest```

<details>
 
<summary> Резуьтат </summary>

```
The push refers to repository [docker.io/pahami/nginxotus]
d94cc1d56f1b: Pushed 
72120687062c: Mounted from library/nginx 
469fc702bc62: Mounted from library/nginx 
74964efcae21: Mounted from library/nginx 
ad4f5bc987ca: Mounted from library/nginx 
ef050c9a03b5: Mounted from library/nginx 
83c20bc61eb8: Mounted from library/nginx 
1024e8977b69: Mounted from library/nginx 
a0904247e36a: Mounted from library/nginx 
latest: digest: sha256:f06c47e6db507966e7900e2214afd41f8b10eff0e0e6b2eb811f74b76fd98c84 size: 2196
```

</details>

Ссылка на загруженный образ в Docker HUB
https://hub.docker.com/repository/docker/pahami/nginxotus/general

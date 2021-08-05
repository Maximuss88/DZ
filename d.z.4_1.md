Задача 1
В данном задании вы научитесь изменять существующие Dockerfile, адаптируя их под нужный инфраструктурный стек.
Измените базовый образ предложенного Dockerfile на Arch Linux c сохранением его функциональности.

FROM ubuntu:latest
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository ppa:vincent-c/ponysay && \
    apt-get update
RUN apt-get install -y ponysay
ENTRYPOINT ["/usr/bin/ponysay"]
CMD ["Hey, netology”]

Для получения зачета, вам необходимо предоставить:
    Написанный вами Dockerfile
    Скриншот вывода командной строки после запуска контейнера из вашего базового образа
    Ссылку на образ в вашем хранилище docker-hub

 
 
Так и не смог найти для пакетного менеджера pacman (используемого в Arch Linux), как добавить репозиторий для ponysay, поэтому использовал образец на основе Alpine Linux:

FROM alpine:latest

LABEL maintainer "author Martijn Pepping, edit by Maks Plaksin"

RUN apk update && \
    apk add openssl python3 texinfo && \
    wget -O ponysay.zip http://github.com/erkin/ponysay/archive/master.zip && \
    unzip ponysay.zip && \
    cd ponysay-master && \
    ./setup.py install --freedom=partial && \
    apk del openssl texinfo && \
    rm -rf /ponysay.zip /ponysay-master /usr/lib/python*/__pycache__/*.pyc /var/cache/apk/*

ENTRYPOINT ["/usr/bin/ponysay"]

Создаем новый образ из докерфайла:
[max@max_centos docker]$ docker build -t ponysay -f dockerfile1 .
……………………
Successfully built 0f8de2752959
Successfully tagged ponysay:latest
[max@max_centos docker]$ docker images
REPOSITORY       TAG         IMAGE ID             CREATED     SIZE
ponysay          latest      0f8de2752959     3 minutes ago   53.2MB
alpine           latest      d4ff818577bc     6 weeks ago     5.6MB
httpd2           latest      0045033d496a     3 months ago    138MB
maximuss88/test  httpd2      0045033d496a     3 months ago    138MB
httpd            latest      0b932df43057     3 months ago    138MB
debian           latest      0d587dfbc4f4     3 months ago    114MB
hello-world      latest      d1165f221234     4 months ago    13.3kB
centos           latest      300e315adb2f     7 months ago    209MB
    
Запускаем:
[max@max_centos docker]$ docker run -it --name Pony ponysay bash
Получилось что-то цветное (скриншот), напоминающее лошадь на сцене))))

Теперь загрузим образ в докерхаб:
[max@max_centos docker]$ docker login -u maximuss88
[max@max_centos docker]$ docker tag ponysay maximuss88/test:ponysay
[max@max_centos docker]$ docker images
REPOSITORY        TAG       IMAGE ID       CREATED             SIZE
ponysay           latest    0f8de2752959   About an hour ago   53.2MB
..............
[max@max_centos docker]$ docker push maximuss88/test:ponysay

https://hub.docker.com/layers/160729654/maximuss88/test/ponysay/images/sha256-cb416707d8cf6b0be12bdfe269bd8e46163abd01c70614de598c12a0689c7449?context=repository
    
**********
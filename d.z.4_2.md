**********
Задача 2
В данной задаче вы составите несколько разных Dockerfile для проекта Jenkins, опубликуем образ в dockerhub.io и посмотрим логи этих контейнеров.
    Составьте 2 Dockerfile:
        Общие моменты:
            Образ должен запускать Jenkins server
        Спецификация первого образа:
            Базовый образ - amazoncorreto
            Присвоить образу тэг ver1
        Спецификация второго образа:
            Базовый образ - ubuntu:latest
            Присвоить образу тэг ver2
    Соберите 2 образа по полученным Dockerfile
    Запустите и проверьте их работоспособность
    Опубликуйте образы в своём dockerhub.io хранилище
Для получения зачета, вам необходимо предоставить:
    Наполнения 2х Dockerfile из задания
    Скриншоты логов запущенных вами контейнеров (из командной строки)
    Скриншоты веб-интерфейса Jenkins запущенных вами контейнеров (достаточно 1 скриншота на контейнер)
    Ссылки на образы в вашем хранилище docker-hub

    
К сожалению, так и не удалось поднять дженкинс в контейнере с ubuntu и amazoncorreto, получилось только запустить его из готового образа и из образа centos:

FROM ubuntu:latest
RUN apt install -y wget
RUN apt install -y gnupg
CMD wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
CMD sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
RUN apt update
RUN apt install -y jenkins
RUN apt install -y systemctl
EXPOSE 8080
CMD systemctl start jenkins

Создаем новый контейнер и запускаем
docker build -t ub_jenk -f dockerfile2 .
docker run -it --name UB_J ub_jenk -p 8080:8080 /bin/bash

Но так и не получилось запустить дженкинс, даже пересоздав контейнер и продублировав все команды вручную (наверняка не хватает каких-то компонентов, контейнерные сборки сильно урезаны) 
root@bca546bcccc5:/# systemctl start jenkins
root@bca546bcccc5:/# systemctl status jenkins
jenkins.service - Controls Jenkins Automation Server
    Loaded: loaded (/etc/init.d/jenkins, disabled)
    Active: failed (dead)
Скриншот логов вложил.

Зато отлично запустился дженкинс из готового образа (скриншоты логов и веб-интерфейса)
docker pull registry.hub.docker.com/zaiste/jenkins
docker tag registry.hub.docker.com/zaiste/jenkins jenkins-image
docker run -it -p 49001:8080 -v /home/max/docker/jenkins:/var/lib/jenkins jenkins-image

Также попробовал запустить дженкинс в контейнере с CentOS 8 (команды ниже), оказалось, в контейнере не запущены systemd или init, поэтому удалось установить дженкинс, но запустить ни через systemctl, ни через service start не вышло. 
Получилось только запустить вручную (скриншот 13)
[root@8c986ee03a82 /]# java -jar /usr/lib/jenkins/jenkins.war

FROM centos
RUN dnf install -y java-11-openjdk-devel.x86_64
CMD rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
CMD mkdir /etc/yum/repos.d/
CMD cd /etc/yum/repos.d/
CMD curl -O https://pkg.jenkins.io/redhat-stable/jenkins.repo
RUN dnf install -y jenkins
EXPOSE 8080
RUN java -jar /usr/lib/jenkins/jenkins.war

Контейнер был запущен
[max@max_centos docker]$ docker run -it -p 10001:8080 --name CentosJenk centosjenk /bin/bash
Затем коммит, создание тега, загрузка на докерхаб
https://hub.docker.com/layers/162739434/maximuss88/test/centosjenk/images/sha256-583941eafaa0ce1c56085218646fd61448ae5f4fedfa475d1b6799a589481bc6?context=repo

Таким образом, получилось дважды запустить дженкинс, из готового образа jenkins (скриншот 10) и из centos (скриншот 13), однако во втором случае пришлось установить вручную и поднимать процесс тоже вручную


**********
Задача 3
В данном задании вы научитесь:
    объединять контейнеры в единую сеть
    исполнять команды "изнутри" контейнера
Для выполнения задания вам нужно:
    Написать Dockerfile:
        Использовать образ https://hub.docker.com/_/node как базовый
        Установить необходимые зависимые библиотеки для запуска npm приложения https://github.com/simplicitesoftware/nodejs-demo
        Выставить у приложения (и контейнера) порт 3000 для прослушки входящих запросов
        Соберите образ и запустите контейнер в фоновом режиме с публикацией порта
    Запустить второй контейнер из образа ubuntu:latest
    Создайть docker network и добавьте в нее оба запущенных контейнера
    Используя docker exec запустить командную строку контейнера ubuntu в интерактивном режиме
    Используя утилиту curl вызвать путь / контейнера с npm приложением
Для получения зачета, вам необходимо предоставить:
    Наполнение Dockerfile с npm приложением
    Скриншот вывода вызова команды списка docker сетей (docker network cli)
    Скриншот вызова утилиты curl с успешным ответом


В этом задании тоже возникли проблемы: скачал и запустил node, но не получилось настроить npm в предложенном примере (команды ниже, возможно причина в устаревшей версии программы), ошибку при установке удалось победить, ошибку запуска нет((

mkdir /usr/app
cd /usr/app
npm install
mv package-lock.json package.json
npm install

root@5589d5226dba:/usr/app# npm install
up to date, audited 1 package in 989ms
found 0 vulnerabilities

root@5589d5226dba:/usr/app# npm start
npm ERR! Missing script: "start"
........

Поэтому решил использовать вместо этого FTP-сервер на моем любимом CentOS))

Поднял vsftpd-3.0.3-33 в контейнере с centos и поднял второй контейнер с ubuntu:
[max@max_centos ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                         NAMES
f9623cbd9a9b   ubuntu    "/bin/bash"              23 seconds ago   Up 22 seconds                                                 test_ftp
5d8724cccc04   centos    "/bin/bash"              11 minutes ago   Up 11 minutes   0.0.0.0:30000->21/tcp, :::30000->21/tcp       FTP

Затам создал сеть и добавил 
[max@max_centos ~]$ docker network create netology
84648ad55e37bc0d25b811f82d3a4cc942fe3b7fcc70e32edc67904243e4d5b2

[max@max_centos ~]$ docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
b2b2fd28b5de   bridge     bridge    local
eea8ad519f86   host       host      local
84648ad55e37   netology   bridge    local
4fe8240d4db5   none       null      local

[max@max_centos ~]$ docker network connect netology FTP
[max@max_centos ~]$ docker network connect netology test_ftp

[max@max_centos ~]$ docker network inspect netology   (довольно длинный вывод, вместо нескольких скриншотов приложу здесь просто текст из консоли, сокращенный)
[
    {
        "Name": "netology",
        "Id": "84648ad55e37bc0d25b811f82d3a4cc942fe3b7fcc70e32edc67904243e4d5b2",
        "Created": "2021-08-15T18:21:39.322435189+04:00",
……………………
"Containers": {
            "5d8724cccc04e6857bea84a797a3282c4949191994de6be4d9bb4a1a74f7a770": {
                "Name": "FTP",
                "EndpointID": "b09b16703c8b7ae49d55f32f29cdb1701f53528aab3ba7fdeec10a3bc6289886",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "f9623cbd9a9ba045dded50ca2eda49c03b24d7f6d3dd507df41d79a850e1f94a": {
                "Name": "test_ftp",
                "EndpointID": "65e203fb59cd8efc29fa7b116914d8ccae103f746653618a8cc6afe7c80ad12a",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
………………………………

И затем проверил подключение с ubuntu на centos с vsftpd (успешно, открывается пустой корневой каталог ftp)
root@f9623cbd9a9b:/# curl ftp://172.18.0.2
drwxr-xr-x    2 0        0               6 Nov 16  2020 pub

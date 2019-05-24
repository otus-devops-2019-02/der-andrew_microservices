# der-andrew_microservices
der-andrew microservices repository

## Настройка интеграции с travis-ci по аналогии с репозиторием infra
- git clone git@github.com:otus-devops-2019-02/der-andrew_microservices.git
- cd der-andrew_microservices/
- mkdir .github
- wget http://bit.ly/otus-pr-template -O .github/PULL_REQUEST_TEMPLATE.md
- git commit -am 'add pull template .github/PULL_REQUEST_TEMPLATE.md'
- git checkout -b docker-1
- wget https://bit.ly/otus-travis-yaml-2019-02 -O .travis.yml
- git commit -am 'Add PR template and travis checks'
- git push -u origin docker-1
- Go to slack channel DevOps team\andrey_zhalnin and say:

'/github subscribe Otus-DevOps-2019-02/<GITHUB_USER>_infra commits:all'
- After git push we'v got message in slack! All right!
- прощлый репозиторий мне так и не удалось подключить к трэвису. в этот раз, кроме строчки из ДЗ `travis encrypt "devops-team-otus:**********" --add notifications.slack.rooms --com` я подключил строчку, предложенную самим трэвисом `travis encrypt "devops-team-otus:**********" --add notifications.slack --com` и.... О чудо! Трэвис заговорил! Что явилось серебряной пулей сказать трудно.

## Start Docker!
- Install docker-machine
```
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```
- sudo bash. Т.к. докер не запускается без рута.
- docker run -it ubuntu:16.04 /bin/bash
- docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}"
- docker run -dt nginx:latest
- docker kill $(docker ps -q)
- docker rm $(docker ps -a -q)

# Docker-2

- Create project ID: docker-239319
- Launch gcloud init
- Create auth file (`$HOME/.config/gcloud/application_default_credentials.json`) for docker-machine:
  + `gcloud auth application-default login`
- export GOOGLE_PROJECT=docker-239319
- Create VMachine:
```
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
docker-host
```
- docker-machine ls                                                 NAME          ACTIVE   DRIVER   STATE     URL                       SWARM   DOCKER     ERRORS
docker-host   -        google   Running   tcp://34.76.75.218:2376           v18.09.5
- eval $(docker-machine env docker-host)
- wget "https://raw.githubusercontent.com/express42/otus-snippets/master/hw-15/mongod.conf" -O mongod.conf
- wget "https://raw.githubusercontent.com/express42/otus-snippets/master/hw-15/start.sh" -O start.sh
- echo DATABASE_URL=127.0.0.1 > db_config
- wget "https://raw.githubusercontent.com/express42/otus-snippets/master/hw-15/Dockerfile" -O Dockerfile
- docker build -t reddit:latest .
- docker images -a
```REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              da6a496c8e4d        2 minutes ago       681MB
reddit              latest              f90fd636d015        2 minutes ago       681MB
<none>              <none>              dcdab4ee70c8        2 minutes ago       681MB
<none>              <none>              93ca68a58045        2 minutes ago       640MB
<none>              <none>              b63b40c57dfb        2 minutes ago       640MB
<none>              <none>              bc763856f9f5        2 minutes ago       640MB
<none>              <none>              5ec700475c03        2 minutes ago       640MB
<none>              <none>              904dae1720de        2 minutes ago       640MB
<none>              <none>              f43551750187        3 minutes ago       637MB
<none>              <none>              0b4af9bbc1f2        3 minutes ago       144MB
ubuntu              16.04               a3551444fc85        4 days ago          119MB
```
- docker run --name reddit -d --network=host reddit:latest
- docker-machine ls
```NAME          ACTIVE   DRIVER   STATE     URL                       SWARM   DOCKER     ERRORS
docker-host   *        google   Running   tcp://34.76.75.218:2376           v18.09.5
```
- Create VPC rule for puma:
```gcloud compute firewall-rules create reddit-app \
--allow tcp:9292 \
--target-tags=docker-machine \
--description="Allow PUMA connections" \
--direction=INGRESS
```
- Successful connect!
http://34.76.75.218:9292/


## Docker hub: регистрация

- Login succeeded
- docker tag reddit:latest avzhalnin/otus-reddit:1.0
- docker push avzhalnin/otus-reddit:1.0
- docker run --name reddit -d -p 9292:9292 avzhalnin/otus-reddit:1.0
- It works!!!
http://localhost:9292/
- docker inspect avzhalnin/otus-reddit:1.0 -f '{{.ContainerConfig.Cmd}}'
`[/bin/sh -c #(nop)  CMD ["/start.sh"]]`
- docker exec -it reddit bash
```root@e33bf6868203:/# mkdir /tmp/1111
root@e33bf6868203:/# exit
```
- docker diff reddit
```
C /root
A /root/.bash_history
C /tmp
A /tmp/1111
A /tmp/mongodb-27017.sock
C /var
C /var/lib
C /var/lib/mongodb
A /var/lib/mongodb/_tmp
A /var/lib/mongodb/journal
A /var/lib/mongodb/journal/j._0
A /var/lib/mongodb/journal/lsn
A /var/lib/mongodb/journal/prealloc.1
A /var/lib/mongodb/journal/prealloc.2
A /var/lib/mongodb/local.0
A /var/lib/mongodb/local.ns
A /var/lib/mongodb/mongod.lock
C /var/log
A /var/log/mongod.log
```
- Clean:
```
docker-machine rm docker-host
eval $(docker-machine env --unset)
```

# Docker-образы Микросервисы
- Create VMachine:
```
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
docker-host
```
- docker-machine ls
```
NAME          ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER     ERRORS
docker-host   -        google   Running   tcp://35.240.103.79:2376           v18.09.6   
```
- eval $(docker-machine env docker-host)
- Мы в корне репозитория: pwd
`/home/andrew/work/OTUS-201902-git/der-andrew_microservices`
- Пересоздаём образы, т.к. всё почикали в прошлый раз)))
- cd docker-monolith
- docker build -t reddit:latest .
- docker run --name reddit -d --network=host reddit:latest
- Работает!!!
http://35.240.103.79:9292/
- docker kill reddit
- docker rm reddit -v
- Продолжаем выполнение дз.
- wget 'https://github.com/express42/reddit/archive/microservices.zip'
- unzip microservices.zip
- rm -f microservices.zip
- mv reddit-microservices src
- cd src
- Create post-py/Dockerfile
```
FROM python:3.6.0-alpine

WORKDIR /app
ADD . /app
RUN apk add --no-cache build-base gcc
RUN pip install -r /app/requirements.txt

ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

ENTRYPOINT ["python3", "post_app.py"]
```
- Create comment/Dockerfile
```
FROM ruby:2.2
RUN apt-get update -qq && apt-get install -y build-essential

ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

ADD Gemfile* $APP_HOME/
RUN bundle install
ADD . $APP_HOME

ENV COMMENT_DATABASE_HOST comment_db
ENV COMMENT_DATABASE comments

CMD ["puma"]
```
- Create ui/Dockerfile
```
FROM ruby:2.2
RUN apt-get update -qq && apt-get install -y build-essential

ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME
ADD Gemfile* $APP_HOME/
RUN bundle install
ADD . $APP_HOME

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
```
- docker pull mongo:latest
- docker build -t avzhalnin/post:1.0 ./post-py
```
<...>
Successfully built b4d01c05fcd3
Successfully tagged avzhalnin/post:1.0
```
- docker build -t avzhalnin/comment:1.0 ./comment
```
ERROR!!!
Failed to fetch http://deb.debian.org/debian/dists/jessie-updates/InRelease  Unable to find expected entry 'main/binary-amd64/Packages' in Release file
```
- Cure:
```
git diff --color comment/Dockerfile
diff --git a/src/comment/Dockerfile b/src/comment/Dockerfile
index 332bb55..31cd4fb 100644
--- a/src/comment/Dockerfile
+++ b/src/comment/Dockerfile
@@ -1,4 +1,5 @@
 FROM ruby:2.2
+RUN printf "deb http://archive.debian.org/debian/ jessie main\n\
+deb-src http://archive.debian.org/debian/ jessie main\n\
+deb http://security.debian.org jessie/updates main\n\
+deb-src http://security.debian.org jessie/updates main" > /etc/apt/sources.list
 RUN apt-get update -qq && apt-get install -y build-essential
 
 ENV APP_HOME /app
```
- Same with ui:
```
git diff --color ui/Dockerfile
diff --git a/src/ui/Dockerfile b/src/ui/Dockerfile
index 3e02445..a5ea06d 100644
--- a/src/ui/Dockerfile
+++ b/src/ui/Dockerfile
@@ -1,4 +1,8 @@
 FROM ruby:2.2
+RUN printf "deb http://archive.debian.org/debian/ jessie main\n\
+deb-src http://archive.debian.org/debian/ jessie main\n\
+deb http://security.debian.org jessie/updates main\n\
+deb-src http://security.debian.org jessie/updates main" > /etc/apt/sources.list
 RUN apt-get update -qq && apt-get install -y build-essential
 
 ENV APP_HOME /app
```
- docker build -t avzhalnin/ui:1.0 ./ui
```
<...>
Successfully built 9c54ca0b4ffc
Successfully tagged avzhalnin/ui:1.0
```
- docker network create reddit
- Запускаем приложение:
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post avzhalnin/post:1.0
docker run -d --network=reddit --network-alias=comment avzhalnin/comment:1.0
docker run -d --network=reddit -p 9292:9292 avzhalnin/ui:1.0
```
- It works again!!!
http://35.240.103.79:9292/


## Задание со звездой

- Kill app:
`docker kill $(docker ps -q)`
- Улучшаем ui:
```
FROM ubuntu:16.04
RUN apt-get update \
    && apt-get install -y ruby-full ruby-dev build-essential \
    && gem install bundler --no-ri --no-rdoc

ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME
ADD Gemfile* $APP_HOME/
RUN bundle install
ADD . $APP_HOME

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
```
- Rebuild ui:
```
docker build -t avzhalnin/ui:2.0 ./ui
<...>
Successfully built ab1774f20a4a
Successfully tagged avzhalnin/ui:2.0

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
avzhalnin/ui        2.0                 ab1774f20a4a        28 seconds ago      449MB
```
- Build ui from alpine and run app:
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post avzhalnin/post:1.0
docker run -d --network=reddit --network-alias=comment avzhalnin/comment:1.0
docker run -d --network=reddit -p 9292:9292 avzhalnin/ui:3.0
```
- App is going fine
http://35.240.103.79:9292/

### Уменьшаем размер образа
- Clean them all!!!
`docker kill $(docker ps -q) && docker rm -v $(docker ps -aq)`
- Make multistage build ui and run app:
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db --name=post_db mongo:latest
docker run -d --network=reddit --network-alias=post --name=post avzhalnin/post:1.0
docker run -d --network=reddit --network-alias=comment --name=comment avzhalnin/comment:1.0
docker run -d --network=reddit -p 9292:9292 --name=ui avzhalnin/ui:4.0
```
- It works!!!
http://35.240.103.79:9292/
- Make multistage build comment and run app.
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db --name=post_db mongo:latest
docker run -d --network=reddit --network-alias=post --name=post avzhalnin/post:1.0
docker run -d --network=reddit --network-alias=comment --name=comment avzhalnin/comment:2.0
docker run -d --network=reddit -p 9292:9292 --name=ui avzhalnin/ui:4.0
```
- It works!!!
http://35.240.103.79:9292/
- Make multistage build post-py and run app.
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db --name=post_db mongo:latest
docker run -d --network=reddit --network-alias=post --name=post avzhalnin/post:2.0
docker run -d --network=reddit --network-alias=comment --name=comment avzhalnin/comment:2.0
docker run -d --network=reddit -p 9292:9292 --name=ui avzhalnin/ui:4.0
- It works!!!
http://35.240.103.79:9292/

## Создание вольюма
- docker volume create reddit_db
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db --name=post_db \
-v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post --name=post avzhalnin/post:2.0
docker run -d --network=reddit --network-alias=comment --name=comment avzhalnin/comment:2.0
docker run -d --network=reddit -p 9292:9292 --name=ui avzhalnin/ui:4.0
```
- Посты на месте!
http://35.240.103.79:9292/post/5cd5eefdcc58bc000ea1a234


# Docker: сети, docker-compose

- Run `docker run -ti --rm --network none joffotron/docker-net-tools -c ifconfig`
```
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
- Run `docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig`
```
br-88e21dad0e20 Link encap:Ethernet  HWaddr 02:42:C9:2D:9B:49  
          inet addr:172.18.0.1  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker0   Link encap:Ethernet  HWaddr 02:42:21:84:27:5C  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

ens4      Link encap:Ethernet  HWaddr 42:01:0A:84:00:06  
          inet addr:10.132.0.6  Bcast:10.132.0.6  Mask:255.255.255.255
          inet6 addr: fe80::4001:aff:fe84:6%32525/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1460  Metric:1
          RX packets:2315 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1753 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:43266553 (41.2 MiB)  TX bytes:183277 (178.9 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1%32525/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
- Run `docker-machine ssh docker-host ifconfig`. Output is the same.
- Run `docker run --network host -d nginx` 4 раза. Тольк первый контейнер запущен, т.к. порт занят для остальных.
- Остановили всех `docker kill $(docker ps -q)`
- Run `docker-machine ssh docker-host sudo ln -s /var/run/docker/netns /var/run/netns`.
- Теперь можно просматривать существующие в данный момент net-namespaces с помощью команды:
`docker-machine ssh docker-host sudo ip netns`
- Примечание: ip netns exec <namespace> <command> - позволит выполнять команды в выбранном namespace.

## Bridge network driver

- Run `docker network create reddit --driver bridge`
- Запустили контейнеры с алиасами и всё заработало
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post avzhalnin/post:1.0
docker run -d --network=reddit --network-alias=comment  avzhalnin/comment:1.0
docker run -d --network=reddit -p 9292:9292 avzhalnin/ui:1.0
```
http://35.240.103.79:9292/
- Разделим сети front end и back end.
- Создадим сети:
```
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```
- И запустим приложение.
```
docker run -d --network=front_net -p 9292:9292 --name ui  avzhalnin/ui:1.0
docker run -d --network=back_net --name comment  avzhalnin/comment:1.0
docker run -d --network=back_net --name post  avzhalnin/post:1.0
docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest
```
- Не всё работает. Подключим контейнеры ко второй сети:
```
docker network connect front_net post
docker network connect front_net comment
```
- Теперь работает как надо.

## Bridge network driver

- Install bridge-utils on docker-host:
`sudo apt-get update && sudo apt-get install bridge-utils`
- List network:
```
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
2055c4603b8f        back_net            bridge              local
d1fb3aee3aad        bridge              bridge              local
9efbe4af6de4        front_net           bridge              local
cac42989932c        host                host                local
1d925b9ebf94        none                null                local
88e21dad0e20        reddit              bridge              local
```
- List bridge:
```
brctl show
bridge name     bridge id               STP enabled     interfaces
br-2055c4603b8f         8000.024297c185d8       no              veth0c7081d
                                                        veth80eb4ef
                                                        veth9e2b985
br-88e21dad0e20         8000.0242106a9006       no
br-9efbe4af6de4         8000.02426bfe3c08       no              veth5e51ba1
                                                        vetha9d4544
                                                        vethc64e84e
docker0         8000.0242ca46e495       no
```
- List iptables:
```
iptables -nvL -t nat
Chain PREROUTING (policy ACCEPT 134 packets, 8290 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   12  4319 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 2 packets, 76 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 4 packets, 256 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 112 packets, 6736 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
   24  1734 MASQUERADE  all  --  *      !br-9efbe4af6de4  10.0.1.0/24          0.0.0.0/0           
    0     0 MASQUERADE  all  --  *      !br-88e21dad0e20  172.18.0.0/16        0.0.0.0/0           
    0     0 MASQUERADE  all  --  *      !br-2055c4603b8f  10.0.2.0/24          0.0.0.0/0           
    0     0 MASQUERADE  tcp  --  *      *       10.0.1.2             10.0.1.2             tcp dpt:9292

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  br-9efbe4af6de4 *       0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  br-88e21dad0e20 *       0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  br-2055c4603b8f *       0.0.0.0/0            0.0.0.0/0           
    0     0 DNAT       tcp  --  !br-9efbe4af6de4 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:9292 to:10.0.1.2:9292
```
- List proxy:
```
pgrep -a docker-proxy
4147 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 9292 -container-ip 10.0.1.2 -container-port 9292
```

## Docker-compose
- Make docker-compose.yml
```
version: '3.3'
services:
  post_db:
    image: mongo:3.2
    volumes:
      - post_db:/data/db
    networks:
      - reddit
  ui:
    build: ./ui
    image: ${USERNAME}/ui:1.0
    ports:
      - 9292:9292/tcp
    networks:
      - reddit
  post:
    build: ./post-py
    image: ${USERNAME}/post:1.0
    networks:
      - reddit
  comment:
    build: ./comment
    image: ${USERNAME}/comment:1.0
    networks:
      - reddit

volumes:
  post_db:

networks:
  reddit:
```
- Kiil them all!
`docker kill $(docker ps -q) && docker rm -v $(docker ps -aq)`
- Run app:
```
export USERNAME=avzhalnin
docker-compose up -d
docker-compose ps
```
- App work good.
http://35.240.103.79:9292/
- Add .env file
- App work fine.
http://35.240.103.79:9292/
- Базовое имя проекта можно задать двумя способами:
1. Через переменную `COMPOSE_PROJECT_NAME`.
2. Использую ключ `-p` docker-compose.

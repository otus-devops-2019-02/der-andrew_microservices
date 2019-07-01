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
- прошлый репозиторий мне так и не удалось подключить к трэвису. в этот раз, кроме строчки из ДЗ `travis encrypt "devops-team-otus:**********" --add notifications.slack.rooms --com` я подключил строчку, предложенную самим трэвисом `travis encrypt "devops-team-otus:**********" --add notifications.slack --com` и.... О чудо! Трэвис заговорил! Что явилось серебряной пулей сказать трудно.

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
2. Используя ключ `-p` docker-compose.

# Устройство Gitlab-CI

## Инсталляция Gitlab CI

- Указываем в каком проекте создавать машину и правила файрвола:

`export GOOGLE_PROJECT=docker-239319`
- Create VMachine:
```
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-disk-size 100 \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
--google-tags http-server,https-server \
gitlab-ci
```
- Create VPC firewall rules http,https:
```
gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

gcloud compute firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server
```
- Настроим окружение для docker-machine:

`eval $(docker-machine env gitlab-ci)`

## Подготавливаем окружение gitlab-ci

- Login to vmachine:
```
docker-machine ssh gitlab-ci
```
- Create tree and docker-compose.yml
```
sudo su
apt-get install -y docker-compose
mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
cd /srv/gitlab/
cat << EOF > docker-compose.yml
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://$(curl ifconfig.me)'
  ports:
    - '80:80'
    - '443:443'
    - '2222:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
EOF
```

## Запускаем gitlab-ci
- Login to vmachine:
```
docker-machine ssh gitlab-ci
```
- Run docker-compose.yml
```
docker-compose up -d
```
- Проверяем.
http://34.76.136.75/
- Создали пароль пользователя, группу, проект.
- Добавили ремоут в микросервисы.
```
git remote add gitlab http://34.76.136.75/homework/example.git
git push gitlab gitlab-ci-1
```
- Add pipeline definition in .gitlab-ci.yml file.

## Run Runner.
- Получили токен для runner:

`DgzD4iyAJZxf_YsVPWjy`
- На сервере gitlab-ci запустить раннер, выполнив:
```
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```
- Register runner:

`docker exec -it gitlab-runner gitlab-runner register --run-untagged --locked=false`
```
Runtime platform                                    arch=amd64 os=linux pid=30 revision=5a147c92 version=11.11.1
Running in system-mode.                            
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://34.76.136.75/
Please enter the gitlab-ci token for this runner:
DgzD4iyAJZxf_YsVPWjy
Please enter the gitlab-ci description for this runner:
[f829ea2fce49]: my-runner
Please enter the gitlab-ci tags for this runner (comma separated):
linux,xenial,ubuntu,docker
Registering runner... succeeded                     runner=DgzD4iyA
Please enter the executor: docker-ssh, shell, docker-ssh+machine, docker+machine, kubernetes, docker, docker-windows, parallels, ssh, virtualbox:
docker
Please enter the default Docker image (e.g. ruby:2.1):
alpine:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
- Добавим исходный код reddit в репозиторий:
```
git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
git add reddit/
git commit -m “Add reddit app”
git push gitlab gitlab-ci-1
```
- Добавили тест в пайплайн.

## Dev окружение.

- Изменили пайплайн и добавили окружение.
- Создалось окружение dev.

`http://34.76.136.75/homework/example/environments`

## Staging и Production

- Изменили пайплайн и добавили окружение.
- Создались окружения stage and production.
- Условия и ограничения.
```
  only:
    - /^\d+\.\d+\.\d+/
```
- Без тэга пайплайн запустился без stage and prod.
- Добавляем тег:
```
git commit -a -m 'test: #4 add logout button to profile page'
git tag 2.4.10
git push gitlab gitlab-ci-1 --tags
```
- С тэгами запустился весь пайплайн.

## Динамические окружения

- Добавили джоб и бранч bugfix

## Удаляем VM

- Создали docker-compose.yml для Gilab.
```
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://34.76.136.75'
  ports:
    - '80:80'
    - '443:443'
    - '2222:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```

# Введение в мониторинг. Задание №20

## Подготовка окружения

- Создадим правило фаервола для Prometheus и Puma:
```
gcloud compute firewall-rules create prometheus-default --allow tcp:9090
gcloud compute firewall-rules create puma-default --allow tcp:9292
```
- Создадим Docker хост в GCE и настроим локальное окружение на работу с ним
```
export GOOGLE_PROJECT=docker-239319
# create docker host
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    docker-host

# configure local env
eval $(docker-machine env docker-host)
```
- Запуск Prometheus
```
docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus:v2.1.0
```
- Заходим в UI
http://35.187.32.118:9090/graph
- Stop container!
`docker stop prometheus`

## Переупорядочим структуру директорий
- Выполним:
```
cd home/andrew/work/OTUS-201902-git/der-andrew_microservices
mkdir docker
git mv docker-monolith docker
git mv src/docker-compose.yml docker
git mv src/.env.example docker
mv src/docker-compose.* docker
mv src/.env docker
mkdir monitoring
echo .env > docker/.gitignore
```

## Создание Docker образа
- Create Dockerfile
```
mkdir docker/prometheus
cat << EOF > docker/prometheus/Dockerfile
FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
EOF
```
- Create prometheus.yml
```
---
global:
  scrape_interval: '5s'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets:
        - 'localhost:9090'

  - job_name: 'ui'
    static_configs:
      - targets:
        - 'ui:9292'

  - job_name: 'comment'
    static_configs:
      - targets:
        - 'comment:9292'
```
- В директории prometheus собираем Docker образ:
```
export USER_NAME=avzhalnin
docker build -t $USER_NAME/prometheus .
```
- Build dockers
```
cd home/andrew/work/OTUS-201902-git/der-andrew_microservices
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```
- All work good.
http://35.187.32.118:9292/
http://35.187.32.118:9090/graph
- Add new service node-exporter
- Add new target for prometheus: node-exporter

## завершение задания
- Пушим образы в dockerhub:
```
docker login
for i in ui comment post prometheus; do echo $i; docker push avzhalnin/$i; done
```
- Ссылки:
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/ui
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/comment
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/post
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/prometheus


# №21 Мониторинг приложения и инфраструктуры

- Подготовка окуружения:
```
export GOOGLE_PROJECT=docker-239319

docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
docker-host

eval $(docker-machine env docker-host)
docker-machine ip docker-host
```
- IP адрес нового хоста: `104.155.92.73`.
- Разделим docker compose файлы на приложение и мониторинг.
- Добавим cAdvisor в конфигурацию Prometheus.
- Пересоберем образ Prometheus с обновленной конфигурацией:
```
export USER_NAME=avzhalnin
cd monitoring/prometheus
docker build -t $USER_NAME/prometheus .
```
- Запустим сервисы:
```
cd docker
docker-compose up -d
docker-compose -f docker-compose-monitoring.yml up -d
```
- Создадим правила файрвола VPC:
```
gcloud compute firewall-rules create prometheus-default --allow tcp:9090
gcloud compute firewall-rules create puma-default --allow tcp:9292
gcloud compute firewall-rules create cadvisor-default --allow tcp:8080
```
- Приложение и мониторинго работают кака надо:
Приложение
http://104.155.92.73:9292/
Prometheus
http://104.155.92.73:9090/graph
cAdvisor
http://104.155.92.73:8080/containers/
http://104.155.92.73:8080/metrics
- Добавим сервис Grafana для визуализации метрик Prometheus.
- Создадим правило файрвола VPC для Grafana:
```
gcloud compute firewall-rules create grafana-default --allow tcp:3000
```
- Grafana is works!
http://104.155.92.73:3000/login
- Подключим дашборд:
```
mkdir -p monitoring/grafana/dashboards
wget 'https://grafana.com/api/dashboards/893/revisions/5/download' -O monitoring/grafana/dashboards/DockerMonitoring.json
```
- Добавим информацию о post-сервисе в конфигурацию Prometheus.
- Пересоберём Prometheus:
```
cd monitoring/prometheus
docker build -t $USER_NAME/prometheus .
```
- Пересоздадим нашу Docker инфраструктуру мониторинга:
```
docker-compose -f docker-compose-monitoring.yml down
docker-compose -f docker-compose-monitoring.yml up -d
```
- Добавили графики в дашборд с запросами:
```
rate(ui_request_count{http_status=~"^[45].*"}[1m])
rate(ui_request_count{http_status=~".*"}[1m])
```
- Добавили графики в дашборд с запросами:
```
histogram_quantile(0.95, sum(rate(ui_request_response_time_bucket[5m])) by (le))
```

## Мониторинг бизнесс-логики
- Добавили графики в дашборд с запросами:
```
rate(comment_count[1h])
rate(post_count[1h])
```

## Alerting
- Configure alertmanager for prometheus:
```
mkdir monitoring/alertmanager
cat << EOF > monitoring/alertmanager/Dockerfile
FROM prom/alertmanager:v0.14.0
ADD config.yml /etc/alertmanager/
EOF

cat <<- EOF > monitoring/alertmanager/config.yml
global:
  slack_api_url: 'https://hooks.slack.com/services/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'

route:
  receiver: 'slack-notifications'

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#andrey_zhalnin'
EOF
```
- Build alertmanager
```
USER_NAME=avzhalnin
docker build -t $USER_NAME/alertmanager .
```
- Добавим новый сервис в компоуз файл мониторинга.
- Создадим файл alerts.yml
```
cat <<- EOF > monitoring/prometheus/alerts.yml
groups:
  - name: alert.rules
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: page
      annotations:
        description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute'
        summary: 'Instance {{ $labels.instance }} down'
EOF
```
- Даоавим в Dockerfile Prometheus-а
```
ADD alerts.yml /etc/prometheus/
```
- Добавим информацию о правилах, в конфиг Prometheus `prometheus.yml`
```
rule_files:
  - "alerts.yml"

alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - "alertmanager:9093"
```
- ReBuild prometheus
```
USER_NAME=avzhalnin
docker build -t $USER_NAME/prometheus .
```
- Restart monitoring dockers:
```
docker-compose -f docker-compose-monitoring.yml down
docker-compose -f docker-compose-monitoring.yml up -d
```
- Создадим правила фаервола VPC:
```
gcloud compute firewall-rules create alertmanager-default --allow tcp:9093
```
- Можно зайти по ссылке:
http://104.155.92.73:9093/#/alerts
- Пушим образы в GitHub:
```
docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
docker push $USER_NAME/alertmanager
```
- Links:
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/ui
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/comment
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/post
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/prometheus
https://cloud.docker.com/u/avzhalnin/repository/docker/avzhalnin/alertmanager

# Логирование и распределенная трассировка №23

## Подготовка

- Клонироввали новые сорцы приложения:
```
mkdir _archive
git mv src _archive
git clone git@github.com:express42/reddit.git src
git checkout logging
rm -fdr src/.git
cp -ru --copy-contents _archive/src src; # only newer files!!!
```
- Добавили в **/src/post-py/Dockerfile** установку пакетов gcc и musl-dev
- Собираем все образы:
```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```

## Подготовка окружения

- Создадим докер хост:
```
export GOOGLE_PROJECT=docker-239319

docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-open-port 5601/tcp \
    --google-open-port 9292/tcp \
    --google-open-port 9411/tcp \
    logging

eval $(docker-machine env logging)

docker-machine ip logging; # 35.202.57.233
```

## Логирование Docker контейнеров

- Создадим отдельный compose-файл для нашей системы логирования в папке docker/
```
export USER_NAME=avzhalnin

cat <<- EOF > docker/docker-compose-logging.yml
version: '3'
services:
  fluentd:
    image: \${USER_NAME}/fluentd
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: elasticsearch
    expose:
      - 9200
    ports:
      - "9200:9200"

  kibana:
    image: kibana
    ports:
      - "5601:5601"
EOF

docker-compose -f docker/docker-compose-logging.yml config
```
- Fluentd.
```
mkdir -p logging/fluentd

cat <<- EOF > logging/fluentd/Dockerfile
FROM fluent/fluentd:v0.12
RUN gem install fluent-plugin-elasticsearch --no-rdoc --no-ri --version 1.9.5
RUN gem install fluent-plugin-grok-parser --no-rdoc --no-ri --version 1.0.0
ADD fluent.conf /fluentd/etc
EOF

cat <<- EOF > logging/fluentd/fluent.conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
EOF
```
- Build Fluentd.
```
(cd logging/fluentd && docker build -t $USER_NAME/fluentd .)
```
- Правим `.env` файл, меняем тэги на `logging` и запускаем приложения.
```
cd docker
docker-compose up -d
```
- It works!!!
http://35.202.57.233:9292/
- Добавили драйвер логирования fluentd в post.
```
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: service.post
```
- Перезапустим все сервисиы.
- Добавили фильтр в fluentd:
```
<filter service.post>
  @type parser
  format json
  key_name log
</filter>
```
- Пересоберём и перезапустим.
```
(cd ../logging/fluentd && docker build -t $USER_NAME/fluentd .)
docker-compose -f docker-compose-logging.yml up -d
```

## Неструктруированные логи

- Добавили драйвер логирования fluentd в post.
```
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: service.ui
```
- Перезапустим ui.
- Добавили regex фильтр.
```
<filter service.ui>
  @type parser
  format /\[(?<time>[^\]]*)\]  (?<level>\S+) (?<user>\S+)[\W]*service=(?<service>\S+)[\W]*event=(?<event>\S+)[\W]*(?:path=(?<path>\S+)[\W]*)?request_id=(?<request_id>\S+)[\W]*(?:remote_addr=(?<remote_addr>\S+)[\W]*)?(?:method= (?<method>\S+)[\W]*)?(?:response_status=(?<response_status>\S+)[\W]*)?(?:message='(?<message>[^\']*)[\W]*)?/'
  key_name log
</filter>
```
- Пересоберём и перезапустим.
```
(cd ../logging/fluentd && docker build -t $USER_NAME/fluentd .)
docker-compose -f docker-compose-logging.yml up -d
```
- Добавили ещё Гроку:
```
<filter service.ui>
  @type parser
  key_name log
  format grok
  grok_pattern %{RUBY_LOGGER}
</filter>

<filter service.ui>
  @type parser
  format grok
  grok_pattern service=%{WORD:service} \| event=%{WORD:event} \| request_id=%{GREEDYDATA:request_id} \| message='%{GREEDYDATA:message}'
  key_name message
  reserve_data true
</filter>
```
- Пересоберём и перезапустим.
```
(cd ../logging/fluentd && docker build -t $USER_NAME/fluentd .)
docker-compose -f docker-compose-logging.yml up -d
```

## Задание со звездой. Распределённый трейсинг: Zipkin.
- Добавили сервис zipkin и переменную для включения zipkin в приложениях:
```
    environment:
      - ZIPKIN_ENABLED=${ZIPKIN_ENABLED}
```
- Перестартуем весь балаган:
```
docker-compose -f docker-compose.yml -f docker-compose-logging.yml down
docker-compose -f docker-compose.yml -f docker-compose-logging.yml up -d
```
- Зашли в UI zipkin.
http://35.202.57.233:9411/zipkin/
- Посмотрели спаны приложения.


# Введение в Kubernetes №25

- Создание манифестов.
```
mkdir -p kubernetes/reddit

cat <<- EOF > kubernetes/reddit/post-deployment.yml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: post-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: post
  template:
    metadata:
      name: post
      labels:
        app: post
    spec:
      containers:
      - image: avzhalnin/post
        name: post
EOF
```

## The Hard Way (https://github.com/kelseyhightower/kubernetes-the-hard-way)

### Installing the Client Tools
- Set a default compute region and zone.
```
gcloud config set compute/region us-west1
gcloud config set compute/zone us-west1-c

gcloud config list
```
- The cfssl and cfssljson command line utilities will be used to provision a PKI Infrastructure and generate TLS certificates.
```
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

cfssl version
```
- Install kubectl. The kubectl command line utility is used to interact with the Kubernetes API Server.
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

kubectl version --client
```

### Provisioning Compute Resources
- Create the kubernetes-the-hard-way custom VPC network:
```
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
```
- A subnet must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster. Create the kubernetes subnet in the kubernetes-the-hard-way VPC network:
```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24
```
- Create a firewall rule that allows internal communication across all protocols:
```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0

gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
```
- Kubernetes Public IP Address. Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:
```
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)

gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```
- Kubernetes Controllers. Create three compute instances which will host the Kubernetes control plane:
```
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
done
```
- Create three compute instances which will host the Kubernetes worker nodes:
```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
```
- List the compute instances in your default compute zone:
```
gcloud compute instances list
```
- Configuring SSH Access.
```
gcloud compute ssh controller-0
```

### Provisioning a CA and Generating TLS Certificates.
- Certificate Authority. Generate the CA configuration file, certificate, and private key:
```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
- Client and Server Certificates. In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes admin user.
- Generate the admin client certificate and private key:
```
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```
- The Kubelet Client Certificates. Generate a certificate and private key for each Kubernetes worker node:
```
for instance in worker-0 worker-1 worker-2; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].networkIP)')

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done
```
- The Controller Manager Client Certificate. Generate the kube-controller-manager client certificate and private key:
```
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```
- Generate the kube-proxy client certificate and private key:
```
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```
- Generate the kube-scheduler client certificate and private key:
```
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```
- The Kubernetes API Server Certificate. The kubernetes-the-hard-way static IP address will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will ensure the certificate can be validated by remote clients. Generate the Kubernetes API Server certificate and private key:
```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```
- The Service Account Key Pair. The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the managing service accounts documentation. Generate the service-account certificate and private key:
```
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
```
- Distribute the Client and Server Certificates. Copy the appropriate certificates and private keys to each worker instance:
```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ca.pem ${instance}-key.pem ${instance}.pem ${instance}:~/
done
```
- Copy the appropriate certificates and private keys to each controller instance:
```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem ${instance}:~/
done
```

### Generating Kubernetes Configuration Files for Authentication
- Kubernetes Public IP Address. Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used. Retrieve the kubernetes-the-hard-way static IP address:
```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')
```
- Generate a kubeconfig file for each worker node:
```
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```
- The kube-proxy Kubernetes Configuration File. Generate a kubeconfig file for the kube-proxy service:
```
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```
- The kube-controller-manager Kubernetes Configuration File. Generate a kubeconfig file for the kube-controller-manager service:
```
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```
- The kube-scheduler Kubernetes Configuration File. Generate a kubeconfig file for the kube-scheduler service:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```
- The admin Kubernetes Configuration File. Generate a kubeconfig file for the admin user:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```
- Distribute the Kubernetes Configuration Files. Copy the appropriate kubelet and kube-proxy kubeconfig files to each worker instance:
```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done
```
- Copy the appropriate kube-controller-manager and kube-scheduler kubeconfig files to each controller instance:
```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
```

### Generating the Data Encryption Config and Key.
- Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to encrypt cluster data at rest.
- The Encryption Key. Generate an encryption key:
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
- The Encryption Config File. Create the encryption-config.yaml encryption config file:
```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
- Copy the encryption-config.yaml encryption config file to each controller instance:
```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp encryption-config.yaml ${instance}:~/
done
```

### Bootstrapping the etcd Cluster
- Kubernetes components are stateless and store cluster state in etcd. In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.
- The commands in this lab must be run on each controller instance: controller-0, controller-1, and controller-2. Login to each controller instance using the gcloud command. Example:
```
gcloud compute ssh controller-0
gcloud compute ssh controller-1
gcloud compute ssh controller-2
```
- Download and Install the etcd Binaries. Download the official etcd release binaries from the coreos/etcd GitHub project:
```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
```
- Extract and install the etcd server and the etcdctl command line utility:
```
{
  tar -xvf etcd-v3.3.9-linux-amd64.tar.gz
  sudo mv etcd-v3.3.9-linux-amd64/etcd* /usr/local/bin/
}
```
- Configure the etcd Server.
```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```
- The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:
```
INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
```
- Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:
```
ETCD_NAME=$(hostname -s)
```
- Create the etcd.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Start the etcd Server. Remember to run the above commands on each controller node: controller-0, controller-1, and controller-2!!!
```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```
- Verification. List the etcd cluster members:
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

### Bootstrapping the Kubernetes Control Plane
- In this lab you will bootstrap the Kubernetes control plane across three compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.
- Running commands in parallel with tmux. Remember to run the above commands on each controller node: controller-0, controller-1, and controller-2.!!!
- Create the Kubernetes configuration directory:
```
sudo mkdir -p /etc/kubernetes/config
```
- Download and Install the Kubernetes Controller Binaries. Download the official Kubernetes release binaries:
```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl"
```
- Install the Kubernetes binaries:
```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```
- Configure the Kubernetes API Server
```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem \
    encryption-config.yaml /var/lib/kubernetes/
}
```
- The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:
```
INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
```
- Create the kube-apiserver.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://10.240.0.10:2379,https://10.240.0.11:2379,https://10.240.0.12:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Configure the Kubernetes Controller Manager. Move the kube-controller-manager kubeconfig into place:
```
sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```
- Create the kube-controller-manager.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Configure the Kubernetes Scheduler. Move the kube-scheduler kubeconfig into place:
```
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```
- Create the kube-scheduler.yaml configuration file:
```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
- Create the kube-scheduler.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Start the Controller Services
```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```
- Allow up to 10 seconds for the Kubernetes API Server to fully initialize.
- Enable HTTP Health Checks. A Google Network Load Balancer will be used to distribute traffic across the three API servers and allow each API server to terminate TLS connections and validate client certificates. The network load balancer only supports HTTP health checks which means the HTTPS endpoint exposed by the API server cannot be used. As a workaround the nginx webserver can be used to proxy HTTP health checks. In this section nginx will be installed and configured to accept HTTP health checks on port 80 and proxy the connections to the API server on https://127.0.0.1:6443/healthz.
- The /healthz API server endpoint does not require authentication by default.
- Install a basic web server to handle HTTP health checks:
```
sudo apt-get install -y nginx
cat > kubernetes.default.svc.cluster.local <<EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
{
  sudo mv kubernetes.default.svc.cluster.local \
    /etc/nginx/sites-available/kubernetes.default.svc.cluster.local

  sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
}
sudo systemctl restart nginx
sudo systemctl enable nginx
```
- Verification
```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```
- Test the nginx HTTP health check proxy:
```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```
- RBAC for Kubelet Authorization
- In this section you will configure RBAC permissions to allow the Kubernetes API Server to access the Kubelet API on each worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.
- This tutorial sets the Kubelet --authorization-mode flag to Webhook. Webhook mode uses the SubjectAccessReview API to determine authorization.
- Create the system:kube-apiserver-to-kubelet ClusterRole with permissions to access the Kubelet API and perform most common tasks associated with managing pods:
```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```
- The Kubernetes API Server authenticates to the Kubelet as the kubernetes user using the client certificate as defined by the --kubelet-client-certificate flag.
- Bind the system:kube-apiserver-to-kubelet ClusterRole to the kubernetes user:
```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```
- The Kubernetes Frontend Load Balancer
- In this section you will provision an external load balancer to front the Kubernetes API Servers. The kubernetes-the-hard-way static IP address will be attached to the resulting load balancer.
```
!!!!!!!!!!!!
The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the same machine used to create the compute instances.
```
- Provision a Network Load Balancer
- Create the external load balancer network resources:
```
{
  KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
    --region $(gcloud config get-value compute/region) \
    --format 'value(address)')

  gcloud compute http-health-checks create kubernetes \
    --description "Kubernetes Health Check" \
    --host "kubernetes.default.svc.cluster.local" \
    --request-path "/healthz"

  gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check \
    --network kubernetes-the-hard-way \
    --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 \
    --allow tcp

  gcloud compute target-pools create kubernetes-target-pool \
    --http-health-check kubernetes

  gcloud compute target-pools add-instances kubernetes-target-pool \
   --instances controller-0,controller-1,controller-2

  gcloud compute forwarding-rules create kubernetes-forwarding-rule \
    --address ${KUBERNETES_PUBLIC_ADDRESS} \
    --ports 6443 \
    --region $(gcloud config get-value compute/region) \
    --target-pool kubernetes-target-pool
}
```
- Verification. Retrieve the kubernetes-the-hard-way static IP address:
```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')
```
- Make a HTTP request for the Kubernetes version info:
```
curl --cacert ca.pem https://${KUBERNETES_PUBLIC_ADDRESS}:6443/version
```

### Bootstrapping the Kubernetes Worker Nodes
- In this lab you will bootstrap three Kubernetes worker nodes. The following components will be installed on each node: runc, gVisor, container networking plugins, containerd, kubelet, and kube-proxy.
- Prerequisites. The commands in this lab must be run on each worker instance: worker-0, worker-1, and worker-2. Login to each worker instance using the gcloud command.
- Running commands in parallel with tmux
- Provisioning a Kubernetes Worker Node
- Install the OS dependencies:
```
{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}
```
- The socat binary enables support for the kubectl port-forward command. Download and Install Worker Binaries
```
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.12.0/crictl-v1.12.0-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc-50c283b9f56bb7200938d9e207355f05f79f0d17 \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.2.0-rc.0/containerd-1.2.0-rc.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubelet
```
- Create the installation directories:
```
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```
- Install the worker binaries:
```
{
  sudo mv runsc-50c283b9f56bb7200938d9e207355f05f79f0d17 runsc
  sudo mv runc.amd64 runc
  chmod +x kubectl kube-proxy kubelet runc runsc
  sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/
  sudo tar -xvf crictl-v1.12.0-linux-amd64.tar.gz -C /usr/local/bin/
  sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/
  sudo tar -xvf containerd-1.2.0-rc.0.linux-amd64.tar.gz -C /
}
```
- Configure CNI Networking. Retrieve the Pod CIDR range for the current compute instance:
```
POD_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/pod-cidr)
```
- Create the bridge network configuration file:
```
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```
- Create the loopback network configuration file:
```
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```
- Configure containerd. Create the containerd configuration file:
```
sudo mkdir -p /etc/containerd/
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
    [plugins.cri.containerd.gvisor]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF
```
- Untrusted workloads will be run using the gVisor (runsc) runtime.
- Create the containerd.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```
- Configure the Kubelet
```
{
  sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.pem /var/lib/kubernetes/
}
```
- Create the kubelet-config.yaml configuration file:
```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```
- The resolvConf configuration is used to avoid loops when using CoreDNS for service discovery on systems running systemd-resolved.
- Create the kubelet.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Configure the Kubernetes Proxy
```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```
- Create the kube-proxy-config.yaml configuration file:
```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```
- Create the kube-proxy.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Start the Worker Services
```
{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
}
```
- Remember to run the above commands on each worker node: worker-0, worker-1, and worker-2.
- Verification.
```
!!!!!!!
The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the same machine used to create the compute instances.
```
- List the registered Kubernetes nodes:
```
gcloud compute ssh controller-0 \
  --command "kubectl get nodes --kubeconfig admin.kubeconfig"
```

### Configuring kubectl for Remote Access
- In this lab you will generate a kubeconfig file for the kubectl command line utility based on the admin user credentials.
- Run the commands in this lab from the same directory used to generate the admin client certificates.
- Generate a kubeconfig file suitable for authenticating as the admin user:
```
{
  KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
    --region $(gcloud config get-value compute/region) \
    --format 'value(address)')

  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```
- Check the health of the remote Kubernetes cluster:
```
kubectl get componentstatuses
```

### Provisioning Pod Network Routes
- Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network routes.
- In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.
- There are other ways to implement the Kubernetes networking model.
- Print the internal IP address and Pod CIDR range for each worker instance:
```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute instances describe ${instance} \
    --format 'value[separator=" "](networkInterfaces[0].networkIP,metadata.items[0].value)'
done
```
- Create network routes for each worker instance:
```
for i in 0 1 2; do
  gcloud compute routes create kubernetes-route-10-200-${i}-0-24 \
    --network kubernetes-the-hard-way \
    --next-hop-address 10.240.0.2${i} \
    --destination-range 10.200.${i}.0/24
done
```
- List the routes in the kubernetes-the-hard-way VPC network:
```
gcloud compute routes list --filter "network: kubernetes-the-hard-way"
```

### Deploying the DNS Cluster Add-on
- In this lab you will deploy the DNS add-on which provides DNS based service discovery, backed by CoreDNS, to applications running inside the Kubernetes cluster.
- The DNS Cluster Add-on
- Deploy the coredns cluster add-on:
```
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
```
- List the pods created by the kube-dns deployment:
```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```
- Verification
- Create a busybox deployment:
```
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```
- List the pod created by the busybox deployment:
```
kubectl get pods -l run=busybox
```
- Retrieve the full name of the busybox pod:
```
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```
- Execute a DNS lookup for the kubernetes service inside the busybox pod:
```
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

### Smoke Test
- In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.
- Data Encryption. In this section you will verify the ability to encrypt secret data at rest.
- Create a generic secret:
```
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```
- Print a hexdump of the kubernetes-the-hard-way secret stored in etcd. The etcd key should be prefixed with k8s:enc:aescbc:v1:key1, which indicates the aescbc provider was used to encrypt the data with the key1 encryption key.
```
gcloud compute ssh controller-0 \
  --command "sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
```
- Deployments. In this section you will verify the ability to create and manage Deployments.
- Create a deployment for the nginx web server:
```
kubectl run nginx --image=nginx
```
- List the pod created by the nginx deployment:
```
kubectl get pods -l run=nginx
```
- Port Forwarding. In this section you will verify the ability to access applications remotely using port forwarding.
- Retrieve the full name of the nginx pod:
```
POD_NAME=$(kubectl get pods -l run=nginx -o jsonpath="{.items[0].metadata.name}")
```
- Forward port 8080 on your local machine to port 80 of the nginx pod:
```
kubectl port-forward $POD_NAME 8080:80
```
- In a new terminal make an HTTP request using the forwarding address:
```
curl --head http://127.0.0.1:8080
```
- Logs. In this section you will verify the ability to retrieve container logs.
- Print the nginx pod logs:
```
kubectl logs $POD_NAME
```
- Services. In this section you will verify the ability to expose applications using a Service.
- Expose the nginx deployment using a NodePort service:
```
kubectl expose deployment nginx --port 80 --type NodePort
```
```
!!!!!!
The LoadBalancer service type can not be used because your cluster is not configured with cloud provider integration. Setting up cloud provider integration is out of scope for this tutorial.
```
- Retrieve the node port assigned to the nginx service:
```
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```
- Create a firewall rule that allows remote access to the nginx node port:
```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-nginx-service \
  --allow=tcp:${NODE_PORT} \
  --network kubernetes-the-hard-way
```
- Retrieve the external IP address of a worker instance:
```
EXTERNAL_IP=$(gcloud compute instances describe worker-0 \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')
```
- Make an HTTP request using the external IP address and the nginx node port:
```
curl -I http://${EXTERNAL_IP}:${NODE_PORT}
```
- Untrusted Workloads. This section will verify the ability to run untrusted workloads using gVisor.
- Create the untrusted pod:
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: untrusted
  annotations:
    io.kubernetes.cri.untrusted-workload: "true"
spec:
  containers:
    - name: webserver
      image: gcr.io/hightowerlabs/helloworld:2.0.0
EOF
```
- Verification. In this section you will verify the untrusted pod is running under gVisor (runsc) by inspecting the assigned worker node.
- Verify the untrusted pod is running:
```
kubectl get pods -o wide
```
- Get the node name where the untrusted pod is running:
```
INSTANCE_NAME=$(kubectl get pod untrusted --output=jsonpath='{.spec.nodeName}')
```
- SSH into the worker node:
```
gcloud compute ssh ${INSTANCE_NAME}
```
- List the containers running under gVisor:
```
sudo runsc --root  /run/containerd/runsc/k8s.io list
```
- Get the ID of the untrusted pod:
```
POD_ID=$(sudo crictl -r unix:///var/run/containerd/containerd.sock \
  pods --name untrusted -q)
```
- Get the ID of the webserver container running in the untrusted pod:
```
CONTAINER_ID=$(sudo crictl -r unix:///var/run/containerd/containerd.sock \
  ps -p ${POD_ID} -q)
```
- Use the gVisor runsc command to display the processes running inside the webserver container:
```
sudo runsc --root /run/containerd/runsc/k8s.io ps ${CONTAINER_ID}
```

## Проверить, что kubectl apply -f <filename> проходит по созданным до этого deployment-ам (ui, post, mongo, comment)
```
for pod in comment mongo post ui; do \
 kubectl apply -f ../reddit/$pod-deployment.yml
done;
```
- ... и поды запускаются
```
kubectl get pods
```
```
NAME                                 READY   STATUS    RESTARTS   AGE
busybox-bd8fb7cbd-x5pjh              1/1     Running   0          38m
comment-deployment-df84f5596-grs26   1/1     Running   0          12m
mongo-deployment-6895dffdf4-b7ls6    1/1     Running   0          26s
nginx-dbddb74b8-5wft6                1/1     Running   0          30m
post-deployment-b668dc698-s6z29      1/1     Running   0          12m
ui-deployment-695c9bdb8b-m69z8       1/1     Running   0          12m
untrusted                            1/1     Running   0          20m
```

### Cleaning Up
- Compute Instances. Delete the controller and worker compute instances:
```
gcloud -q compute instances delete \
  controller-0 controller-1 controller-2 \
  worker-0 worker-1 worker-2
```
- Networking. Delete the external load balancer network resources:
```
{
  gcloud -q compute forwarding-rules delete kubernetes-forwarding-rule \
    --region $(gcloud config get-value compute/region)

  gcloud -q compute target-pools delete kubernetes-target-pool

  gcloud -q compute http-health-checks delete kubernetes

  gcloud -q compute addresses delete kubernetes-the-hard-way
}
```
- Delete the kubernetes-the-hard-way firewall rules:
```
gcloud -q compute firewall-rules delete \
  kubernetes-the-hard-way-allow-nginx-service \
  kubernetes-the-hard-way-allow-internal \
  kubernetes-the-hard-way-allow-external \
  kubernetes-the-hard-way-allow-health-check
```
- Delete the kubernetes-the-hard-way network VPC:
```
{
  gcloud -q compute routes delete \
    kubernetes-route-10-200-0-0-24 \
    kubernetes-route-10-200-1-0-24 \
    kubernetes-route-10-200-2-0-24

  gcloud -q compute networks subnets delete kubernetes

  gcloud -q compute networks delete kubernetes-the-hard-way
}
```


# Kubernetes. Запуск кластера и приложения. Модель безопасности. №26

## Installing minikube.
- Install Minikube via direct download:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo install minikube /usr/local/bin
```
- Запустим наш Minukube-кластер.
```
minikube start
```
- Проверяем командой `kubectl get nodes`
```
NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   105s   v1.15.0
```
- Обновили kubectrl.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
sudo mv kubectl /usr/local/bin
```
- Обновили ui-deployment.yml:
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit
      component: ui
  template:
    metadata:
      name: ui-pod
      labels:
        app: reddit
        component: ui
    spec:
      containers:
      - image: avzhalnin/ui
        name: ui
```
- Запустим в Minikube ui-компоненту.
```
kubectl apply -f ui-deployment.yml
```
- Ждём пару минут и проверяем командой `kubectl get deployment`. Должны быть `3`-ки
```
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     3/3     3            3           98s
```
- Найдем, используя selector, POD-ы приложения и пробросим порт.
```
kubectl get pods --selector component=ui
kubectl port-forward --address 0.0.0.0 ui-797f94946b-5qtt6 8080:9292
```
- Проверим работу:
http://10.0.113.200:8080/
- Обновили comment-deployment.yml:
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: comment
  labels:
    app: reddit
    component: comment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit
      component: comment
  template:
    metadata:
      name: comment
      labels:
        app: reddit
        component: comment
    spec:
      containers:
      - image: avzhalnin/comment
        name: comment
```
- Запустим в Minikube компоненту.
```
kubectl apply -f comment-deployment.yml
```
- Пробросили порты и проверили.
http://10.0.113.200:8080/healthcheck
- Обновили post-deployment.yml:
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: post-deployment
  labels:
    app: post
    component: post
spec:
  replicas: 3
  selector:
    matchLabels:
      app: post
      component: post
  template:
    metadata:
      name: post
      labels:
        app: post
        component: post
    spec:
      containers:
      - image: avzhalnin/post
        name: post
```
- Запустим в Minikube компоненту.
```
kubectl apply -f post-deployment.yml
```
- Пробросили порты и проверили. `nc -v 10.0.113.200 5000`
```
Connection to 10.0.113.200 5000 port [tcp/*] succeeded!
```
- Обновили mongo-deployment.yml:
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: reddit
        component: mongo
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongo-persistent-storage
        emptyDir: {}
```
- Запустим в Minikube компоненту.
```
kubectl apply -f mongo-deployment.yml
```
- Создали сервис comment-service.yml
```
---
apiVersion: v1
kind: Service
metadata:
  name: comment
  labels:
    app: reddit
    component: comment
spec:
  ports:
  - port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: comment
```
- Запустим в Minikube компоненту.
```
kubectl apply -f comment-service.yml
```
- Проверили `kubectl describe service comment | grep Endpoints`
```
Endpoints:         172.17.0.7:9292,172.17.0.8:9292,172.17.0.9:9292
```
- Проверили `kubectl exec -it post-76474fb7db-5scm4 nslookup comment`
```
Name:      comment
Address 1: 10.109.118.77 comment.default.svc.cluster.local
```
- Создали сервис post-service.yml
```
---
apiVersion: v1
kind: Service
metadata:
  name: post
  labels:
    app: post
    component: post
spec:
  ports:
  - port: 5000
    protocol: TCP
    targetPort: 5000
  selector:
    app: post
    component: post
```
- Запустим в Minikube компоненту.
```
kubectl apply -f post-service.yml
```
- Проверили `kubectl exec -it ui-797f94946b-5qtt6 nslookup post`
```
Name:   post.default.svc.cluster.local
Address: 10.103.200.32
```
- Создали сервис mongodb-service.yml
```
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  labels:
    app: reddit
    component: mongo
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: reddit
    component: mongo
```
- Запустим в Minikube компоненту.
```
kubectl apply -f mongodb-service.yml
```
- Проверили все сервисы `kubectl get services --show-labels`
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE     LABELS
comment      ClusterIP   10.109.118.77    <none>        9292/TCP    50m     app=reddit,component=comment
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     3h55m   component=apiserver,provider=kubernetes
mongodb      ClusterIP   10.109.100.136   <none>        27017/TCP   54s     app=reddit,component=mongo
post         ClusterIP   10.103.200.32    <none>        5000/TCP    5m31s   app=post,component=post
```
- Пробрасываем порт `kubectl port-forward --address 0.0.0.0 ui-797f94946b-54stz 9292:9292`
- Проверяем
http://10.0.113.200:9292/
- В логах приложение ищет совсем другой адрес: comment_db, а не mongodb. Аналогично и сервис comment ищет post_db. Эти адреса заданы в их Dockerfile-ах в виде переменных окружения.
- В Docker Swarm проблема доступа к одному ресурсу под разными именами решалась с помощью сетевых алиасов. В Kubernetes такого функционала нет. Мы эту проблему можем решить с помощью тех же Service-ов.
- Сделаем Service для БД comment-mongodb-service.yml:
```
---
apiVersion: v1
kind: Service
metadata:
  name: comment-db
  labels:
    app: reddit
    component: mongo
    comment-db: "true"
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: reddit
    component: mongo
    comment-db: "true"
```
- Так же придется обновить файл deployment для mongodb, чтобы новый Service смог найти нужный POD.
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    comment-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: reddit
        component: mongo
        comment-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongo-persistent-storage
        emptyDir: {}
```
- Зададим pod-ам comment переменную окружения для обращения к базе:
```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: comment
  labels:
    app: reddit
    component: comment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit
      component: comment
  template:
    metadata:
      name: comment
      labels:
        app: reddit
        component: comment
    spec:
      containers:
      - image: avzhalnin/comment
        name: comment
        env:
        - name: COMMENT_DATABASE_HOST
          value: comment-db
```
- То же для post.
```
post-mongodb-service.yml
---
apiVersion: v1
kind: Service
metadata:
  name: post-db
  labels:
    app: reddit
    component: mongo
    comment-db: "true"
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: reddit
    component: mongo
    comment-db: "true"

post-deployment.yml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: post
  labels:
    app: post
    component: post
spec:
  replicas: 3
  selector:
    matchLabels:
      app: post
      component: post
  template:
    metadata:
      name: post
      labels:
        app: post
        component: post
    spec:
      containers:
      - image: avzhalnin/post
        name: post
        env:
        - name: POST_DATABASE_HOST
          value: post-db
```
- Пробросили порты `kubectl port-forward --address 0.0.0.0 ui-797f94946b-2wxx9 9292:9292` и проверили. Всё работает!!!
http://10.0.113.200:9292/
- Нам нужно как-то обеспечить доступ к ui-сервису снаружи. Для этого нам понадобится Service для UI-компоненты.
```
---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: NodePort
  ports:
  - nodePort: 32092
    port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
```
- Проверим доступ через адрес виртуалки с minikube.
```
minikube service ui
```
- Работает!
http://192.168.99.100:32092/
- Список сервисов смотрим командой `minikube service list`
```
|-------------|----------------------|-----------------------------|
|  NAMESPACE  |         NAME         |             URL             |
|-------------|----------------------|-----------------------------|
| default     | comment              | No node port                |
| default     | comment-db           | No node port                |
| default     | kubernetes           | No node port                |
| default     | mongodb              | No node port                |
| default     | post                 | No node port                |
| default     | post-db              | No node port                |
| default     | ui                   | http://192.168.99.100:32092 |
| kube-system | kube-dns             | No node port                |
| kube-system | kubernetes-dashboard | No node port                |
|-------------|----------------------|-----------------------------|
```
- Список расширений minikube `minikube addons list`
```
- addon-manager: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- gvisor: disabled
- heapster: disabled
- ingress: disabled
- logviewer: disabled
- metrics-server: disabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled
- storage-provisioner-gluster: disabled
```
- Объекты нашего dashboard `kubectl get all -n kube-system --selector app=kubernetes-dashboard`
```
NAME                                        READY   STATUS    RESTARTS   AGE
pod/kubernetes-dashboard-7b8ddcb5d6-5qtp8   1/1     Running   0          59m
NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes-dashboard   ClusterIP   10.108.116.238   <none>        80/TCP    59m
NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/kubernetes-dashboard-7b8ddcb5d6   1         1         1       59m
```
- Зайдем в Dashboard `minikube service kubernetes-dashboard -n kube-system`

## Namespace
- Создадим свой.
```
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
- Добавим инфу об окружении внутрь контейнера UI
```
        env:
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
```

## Разворачиваем Kubernetes
- Создали кластер:
```
gcloud beta container --project "docker-239319" clusters create "standard-cluster-1" \
  --zone "us-central1-a" --no-enable-basic-auth --cluster-version "1.12.8-gke.10" \
  --machine-type "n1-standard-1" --image-type "COS" --disk-type "pd-standard" --disk-size "20" \
  --scopes "https://www.googleapis.com/auth/devstorage.read_only",\
  "https://www.googleapis.com/auth/logging.write", \
  "https://www.googleapis.com/auth/monitoring",\
  "https://www.googleapis.com/auth/servicecontrol",\
  "https:// www.googleapis.com/auth/service.management.readonly",\
  "https://www.googleapis.com/auth/trace.append" \
  --num-nodes "2" --enable-cloud-logging --enable-cloud-monitoring --no-enable-ip-alias \
  --network "projects/docker-239319/global/networks/default" \
  --subnetwork "projects/docker-239319/regions/us-central1/subnetworks/default" \
  --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair
```
- Подключимся к кластеру (нажать Connect и скопировать команду).
```
gcloud container clusters get-credentials standard-cluster-1 --zone us-central1-a --project docker-239319
```
- Проверяем командой `kubectl config current-context`
- Создадим dev namespace
```
kubectl apply -f reddit/dev-namespace.yml
```
- Задеплоим всё приложение в namespace dev:
```
kubectl apply -n dev -f reddit/
```
- Откроем Reddit для внешнего мира:
```
gcloud compute --project=docker-239319 firewall-rules create gce-cluster-reddit-app-access \
  --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:30000-32767 \
  --source-ranges=0.0.0.0/0
```
- Найдите внешний IP-адрес любой ноды из кластера `kubectl get nodes -o wide`
```
NAME                                                STATUS   ROLES    AGE   VERSION          INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-standard-cluster-1-default-pool-ab2dcd4c-pksp   Ready    <none>   52m   v1.12.8-gke.10   10.128.0.3    35.238.120.12   Container-Optimized OS from Google   4.14.127+        docker://17.3.2
gke-standard-cluster-1-default-pool-ab2dcd4c-sz5s   Ready    <none>   52m   v1.12.8-gke.10   10.128.0.4    35.238.22.205   Container-Optimized OS from Google   4.14.127+        docker://17.3.2
```
- Найдите порт публикации сервиса ui
```
kubectl describe service ui -n dev | grep NodePort
```
- Проверяем:
http://35.238.120.12:32092/
- В кластере включаем addon dashboadd.
- Ждём пока кластер загрузится.
- Даём команду `kubectl proxy`.
- Заходим по адресу:
http://localhost:8001/ui
- Провал, т.к. не хватает прав. 
- Нужно нашему Service Account назначить роль с достаточными правами на просмотр информации о кластере В кластере уже есть объект ClusterRole с названием cluster-admin. Тот, кому назначена эта роль имеет полный доступ ко всем объектам кластера.
- Добавим их.
```
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
```


# Kubernetes. Networks ,Storages. Задание №27

## Load balancer.
- Настроим соответствующим образом Service UI:
```
---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: LoadBalancer
  ports:
  - port: 80
    nodePort: 32092
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
```
- Применяем изменения `kubectl apply -f ui-service.yml -n dev`
- Преверяем `kubectl get service -n dev --selector component=ui`
```
NAME   TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
ui     LoadBalancer   10.11.252.109   35.202.189.62   80:32092/TCP   135m
```
- Работает!
http://35.202.189.62/

## Ingress
- Google в GKE уже предоставляет возможность использовать их собственные решения балансирощик в качестве Ingress controller-ов. Перейдите в настройки кластера раздел Дополнения(add-ons) в веб-консоли gcloud. Убедитесь, что встроенный Ingress включен.
- Создадим Ingress для сервиса UI.
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ui
spec:
  backend:
    serviceName: ui
    servicePort: 80
```
- Применим `kubectl apply -f ui-ingress.yml -n dev`.
- В gke балансировщиках нагрузки появились несколько правил.
- Преоврим в кластере ` kubectl get ingress -n dev`
```
NAME   HOSTS   ADDRESS          PORTS   AGE
ui     *       35.244.196.102   80      4m5s
```
- Через 1-2 минуты проверяем.
http://35.244.196.102/
- Один балансировщик можно спокойно убрать. Обновим сервис для UI.
```
---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: NodePort
  ports:
  - port: 9292
    nodePort: 32092
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
```
- Применим `kubectl apply -f ui-service.yml -n dev`
- Заставим работать Ingress Controller как классический веб. ui-ingress.yml:
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ui
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: ui
          servicePort: 9292
```
- Применяем `kubectl apply -f ui-ingress.yml -n dev`
- Проверяем через 1-2 минуты:
http://35.244.196.102/


## Secret
- Защитим наш сервис с помощью TLS.
- Подготовим сертификат используя IP как CN.
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=35.244.196.102"
```
- Загрузит сертификат в кластер kubernetes.
```
kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev
```
- Проверить можно командой `kubectl describe secret ui-ingress -n dev`.
```
Name:         ui-ingress
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Type:  kubernetes.io/tls
Data
====
tls.crt:  1127 bytes
tls.key:  1704 bytes
```
- TLS Termination. Настроим Ingress на прием только HTTPS траффика. ui-ingress.yml:
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ui
  annotations:
    kubernetes.io/ingress.allow-http: "false"
spec:
  tls:
  - secretName: ui-ingress
  backend:
    serviceName: ui
    servicePort: 9292
```
- Применим `kubectl apply -f ui-ingress.yml -n dev`.
- Проверим, подождав 1-2 минуты:
https://35.244.196.102/



## Network Policy
- Найдите имя кластера `gcloud beta container clusters list`
```
NAME                LOCATION       MASTER_VERSION  MASTER_IP      MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS
standard-cluster-1  us-central1-a  1.12.8-gke.10   35.184.48.221  n1-standard-1  1.12.8-gke.10  2          RUNNING
```
- Включим network-policy для GKE.
```
gcloud beta container clusters update standard-cluster-1 --zone=us-central1-a --update-addons=NetworkPolicy=ENABLED
gcloud beta container clusters update standard-cluster-1 --zone=us-central1-a  --enable-network-policy
```
- mongo-network-policy.yml:
```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-db-traffic
  labels:
    app: reddit
spec:
  podSelector:
    matchLabels:
      app: reddit
      component: mongo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: reddit
          component: comment
```
- Применяем политику `kubectl apply -f mongo-network-policy.yml -n dev`
- Проверяем `kubectl -n dev get networkpolicy`
- 


## Хранилище для базы
- Обновим mongo-deployment.yml:
```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    post-db: "true"
    comment-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: reddit
        component: mongo
        post-db: "true"
        comment-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongo-persistent-storage
        emptyDir: {}
```
- Применили `kubectl apply -f mongo-deployment.yml -n dev`
- Создадим диск в Google Cloud.
```
gcloud compute disks create --size=25GB --zone=us-central1-a reddit-mongo-disk
```
- Добавим новый Volume POD-у базы.
```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    post-db: "true"
    comment-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: reddit
        component: mongo
        post-db: "true"
        comment-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-persistent-storage
        emptyDir: {}
        volumes:
      - name: mongo-gce-pd-storage
        gcePersistentDisk:
          pdName: reddit-mongo-disk
          fsType: ext4
```
- Монтируем выделенный диск к POD’у mongo `kubectl apply -f mongo-deployment.yml -n dev`.
- `!!! пересоздания Pod'а (занимает до 10 минут) !!!`
- После пересоздания mongo, посты сохранены.



## PersistentVolume
- Создадим описание PersistentVolume mongo-volume.yml.
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: reddit-mongo-disk
spec:
  capacity:
    storage: 25Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    fsType: "ext4" 
    pdName: "reddit-mongo-disk"
```
- Добавим PersistentVolume в кластер `kubectl apply -f mongo-volume.yml -n dev`
- Создадим описание PersistentVolumeClaim (PVC) mongo-claim.yml:
```
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 15Gi
```
- Добавим PersistentVolumeClaim в кластер `kubectl apply -f mongo-claim.yml -n dev`
- Проверим `kubectl -n dev get pv`
- Одновременно использовать один PV можно только по одному Claim’у. Если Claim не найдет по заданным параметрам PV внутри кластера, либо тот будет занят другим Claim’ом то он сам создаст нужный ему PV воспользовавшись стандартным StorageClass. `kubectl describe storageclass standard -n dev`
```
Name:                  standard
IsDefaultClass:        Yes
Annotations:           storageclass.beta.kubernetes.io/is-default-class=true
Provisioner:           kubernetes.io/gce-pd
Parameters:            type=pd-standard
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```
- Подключим PVC к нашим Pod'ам.
```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    post-db: "true"
    comment-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: reddit
        component: mongo
        post-db: "true"
        comment-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage
        persistentVolumeClaim:
          claimName: mongo-pvc
```
- Обновим описание нашего Deployment’а `kubectl apply -f mongo-deployment.yml -n dev`


## Динамическое выделение Volume'ов
- Но гораздо интереснее создавать хранилища при необходимости и в автоматическом режиме. В этом нам помогут StorageClass’ы. Они описывают где (какой провайдер) и какие хранилища создаются.
- Создадим описание StorageClass’а storage-fast.yml:
```
---
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
- Добавим StorageClass в кластер `kubectl apply -f storage-fast.yml -n dev`
- Проверим `kubectl -n dev get sc`
- PVC + StorageClass. Создадим описание PersistentVolumeClaim. mongo-claim-dynamic.yml:
```
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongo-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 10Gi
```
- Добавим StorageClass в кластер `kubectl apply -f mongo-claim-dynamic.yml -n dev`
- Подключим PVC к нашим Pod'ам mongo-deployment.yml:
```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    post-db: "true"
    comment-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: reddit
        component: mongo
        post-db: "true"
        comment-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage
        persistentVolumeClaim:
          claimName: mongo-pvc-dynamic
```
- Обновим описание нашего Deployment'а `kubectl apply -f mongo-deployment.yml -n dev`
- Проверяем `kubectl -n dev get pv`
```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS   REASON   AGE
pvc-a9ee6fa2-9a98-11e9-8595-42010a800259   15Gi       RWO            Delete           Bound       dev/mongo-pvc           standard                26m
pvc-e28041eb-9a9b-11e9-8595-42010a800259   10Gi       RWO            Delete           Bound       dev/mongo-pvc-dynamic   fast                    3m11s
reddit-mongo-disk                          25Gi       RWO            Retain           Available                                                   30m
```




# CI/CD в Kubernetes №27

## Helm
- Install Helm.
```
wget 'https://get.helm.sh/helm-v2.14.1-linux-amd64.tar.gz'
tar -xzf helm-v2.14.1-linux-amd64.tar.gz linux-amd64/helm
sudo mv linux-amd64/helm /usr/local/bin
rm -dfr linux-amd64 helm-v2.14.1-linux-amd64.tar.gz
```
- Install Tiller.
```
cat << EOF > tiller.yml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EOF

kubectl apply -f tiller.yml
```
- Запустим tiller-сервер `helm init --service-account tiller`.
- Проверим `kubectl get pods -n kube-system --selector app=helm`.
```
NAME                             READY   STATUS    RESTARTS   AGE
tiller-deploy-7b659b7fbd-xnn62   1/1     Running   0          74s
```

## Charts
- Создайте директорию Charts в папке kubernetes со следующей структурой директорий:
```
kubernetes
  ├──Charts
     ├── comment
     ├── post
     ├── reddit
     └── ui
mkdir Charts
cd Charts
for d in comment post reddit ui; do mkdir $d;done
cd ..
tree Charts
```
- Создание чарта для ui.
```
cd Charts
cat << EOF > ui/Chart.yaml
name: ui
version: 1.0.0
description: OTUS reddit application UI
maintainers:
  - name: AndrewZ
    email: andrewz@gmail.com
appVersion: 1.0
EOF
```
- Создание шаблонов для ui.
```
mkdir ui/templates
for f in deployment ingress service; do git mv ../reddit/ui-$f.yml ui/templates/$f.yaml;done
```
- Установим Chart.
```
helm install --name test-ui-1 ui/
```
- Проверяем `helm ls`
- Шаблонизируем chart-ы.
```
cat << EOF > ui/templates/service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  type: NodePort
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
EOF

cat << EOF > ui/templates/deployment.yaml
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: reddit
      component: ui
      release: {{ .Release.Name }}
  template:
    metadata:
      name: ui
      labels:
        app: reddit
        component: ui
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: avzhalnin/ui
        name: ui
        ports:
        - containerPort: 9292
          name: ui
          protocol: TCP
        env:
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
EOF


cat << EOF > ui/templates/ingress.yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: {{ .Release.Name }}-{{ .Chart.Name }}
          servicePort: 9292
EOF
```
- Определим значения собственных переменных 
```
cat << EOF > ui/values.yaml
---
service:
  internalPort: 9292
  externalPort: 9292

image:
  repository: avzhalnin/ui
  tag: latest
EOF
```
- Установим несколько релизов
```
helm install ui --name ui-1
helm install ui --name ui-2
helm install ui --name ui-3
```
- Должны появиться 3 ingress'а `kubectl get ingress`
- Кастомизируем установку своими переменными (образ и порт).
```
cat << EOF > ui/templates/deployment.yaml
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: reddit
      component: ui
      release: {{ .Release.Name }}
  template:
    metadata:
      name: ui
      labels:
        app: reddit
        component: ui
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: ui
        ports:
        - containerPort: {{ .Values.service.internalPort }}
          name: ui
          protocol: TCP
        env:
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
EOF

cat << EOF > ui/templates/service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  type: NodePort
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
EOF

cat << EOF > ui/templates/ingress.yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: {{ .Release.Name }}-{{ .Chart.Name }}
          servicePort: {{ .Values.service.externalPort }}
EOF
```
- Обновим чарты.
```
helm upgrade ui-1 ui/
helm upgrade ui-2 ui/
helm upgrade ui-3 ui/
```
- Осталось собрать пакеты для остальных компонент.
```
mkdir post/templates

cat << EOF > post/Chart.yaml
name: post
version: 1.0.0
description: OTUS reddit application POST
maintainers:
  - name: AndrewZ
    email: andrewz@gmail.com
appVersion: 1.0
EOF


cat << EOF > post/templates/service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: post
    release: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: post
    release: {{ .Release.Name }}
EOF


cat << EOF > post/templates/deployment.yaml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: post
    release: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: post
      release: {{ .Release.Name }}
  template:
    metadata:
      name: post
      labels:
        app: reddit
        component: post
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: post
        ports:
        - containerPort: {{ .Values.service.internalPort }}
          name: post
          protocol: TCP
        env:
        - name: POST_DATABASE_HOST
          value: {{ .Values.databaseHost | default (printf "%s-mongodb" .Release.Name) }}
EOF


cat << EOF > post/values.yaml
---
service:
  internalPort: 5000
  externalPort: 5000

image:
  repository: avzhalnin/post
  tag: latest

databaseHost:
EOF



mkdir comment/templates

cat << EOF > comment/Chart.yaml
name: comment
version: 1.0.0
description: OTUS reddit application COMMENT
maintainers:
  - name: AndrewZ
    email: andrewz@gmail.com
appVersion: 1.0
EOF


cat << EOF > comment/templates/service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
EOF


cat << EOF > comment/templates/deployment.yaml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: comment
      release: {{ .Release.Name }}
  template:
    metadata:
      name: comment
      labels:
        app: reddit
        component: comment
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: comment
        ports:
        - containerPort: {{ .Values.service.internalPort }}
          name: comment
          protocol: TCP
        env:
        - name: COMMENT_DATABASE_HOST
          value: {{ .Values.databaseHost | default (printf "%s-mongodb" .Release.Name) }}
EOF



cat << EOF > comment/values.yaml
---
service:
  internalPort: 9292
  externalPort: 9292

image:
  repository: avzhalnin/comment
  tag: latest

databaseHost:
EOF
```
- Helper for comment.
```
cat << EOF > comment/templates/_helpers.tpl
{{- define "comment.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end -}}
EOF
```
- Use helper.
```
cat << EOF > comment/templates/service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: {{ template "comment.fullname" . }}
  labels:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
EOF
```
- Helper for post and ui.
```
cat << EOF > post/templates/_helpers.tpl
{{- define "post.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end -}}
EOF

cat << EOF > ui/templates/_helpers.tpl
{{- define "ui.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end -}}
EOF
```


## Управление зависимостями
- Создайте reddit/Chart.yaml
```
cat << EOF > reddit/Chart.yaml
name: reddit
version: 1.0.0
description: OTUS simple reddit application
maintainers:
  - name: AndrewZ
    email: andrewz@gmail.com
appVersion: 1.0
EOF
```
- Создайте пустой reddit/values.yaml
```
touch reddit/values.yaml
```
- В директории Chart'а reddit создадим
```
cat << EOF > reddit/requirements.yaml
dependencies:
  - name: ui
    version: "1.0.0"
    repository: "file://../ui"
  - name: post
    version: 1.0.0
    repository: file://../post
  - name: comment
    version: 1.0.0
    repository: file://../comment
EOF
```
- Нужно загрузить зависимости (когда Chart’ не упакован в tgz архив)
```
helm dep update
```
- Chart для базы данных не будем создавать вручную. Возьмем готовый.
- Найдем Chart в общедоступном репозитории `helm search mongo`
- Обновим зависимости.
```
cat << EOF > reddit/requirements.yaml
dependencies:
  - name: ui
    version: "1.0.0"
    repository: "file://../ui"
  - name: post
    version: 1.0.0
    repository: file://../post
  - name: comment
    version: 1.0.0
    repository: file://../comment
  - name: mongodb
    version: 0.4.18
    repository: https://kubernetes-charts.storage.googleapis.com
EOF
```
- Выгрузим зависимости `helm dep update`
- Установим наше приложение:
```
cd ..
helm install reddit --name reddit-test
```
- Есть проблема с тем, что UI-сервис не знает как правильно ходить в post и comment сервисы. Ведь их имена теперь динамические и зависят от имен чартов В Dockerfile UI-сервиса уже заданы переменные окружения. Надо, чтобы они указывали на нужные бекенды.
- Добавим в ui/deployments.yaml
```
cat << EOF > ui/deployments.yaml
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: reddit
      component: ui
      release: {{ .Release.Name }}
  template:
    metadata:
      name: ui
      labels:
        app: reddit
        component: ui
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: ui
        ports:
        - containerPort: {{ .Values.service.internalPort }}
          name: ui
          protocol: TCP
        env:
        - name: POST_SERVICE_HOST
          value: {{ .Values.postHost | default (printf "%s-post" .Release.Name) }}
        - name: POST_SERVICE_PORT
          value: {{ .Values.postPort | default "5000" | quote }}
        - name: COMMENT_SERVICE_HOST
          value: {{ .Values.commentHost | default (printf "%s-comment".Release.Name) }}
        - name: COMMENT_SERVICE_PORT
          value: {{ .Values.commentPort | default "9292" | quote }}
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
EOF
```
- Создадим reddit/values.yaml
```
cat << EOF > reddit/values.yaml
comment:
  image:
    repository: avzhalnin/comment
    tag: latest
  service:
    externalPort: 9292

post:
  image:
    repository: avzhalnin/post
    tag: latest
    service:
      externalPort: 5000

ui:
  image:
    repository: avzhalnin/ui
    tag: latest
    service:
      externalPort: 9292
EOF
```
- После обновления UI - нужно обновить зависимости чарта reddit. `helm dep update ./reddit`
- Обновите релиз, установленный в k8s `helm upgrade reddit-test ./reddit`
- Проверяем
http://35.244.135.236/


## GitLab+Kubernetes
- Добавили bigpool в кластер с более мощнной виртуалкой.
- Отключите RBAC (в настройках кластера - Устаревшие права доступа Legacy Authorization) для упрощения работы. Gitlab-Omnibus пока не подготовлен для этого, а самим это в рамках аботы смысла делать нет.

## Установим GitLab
- Добавим репозиторий Gitlab `helm repo add gitlab https://charts.gitlab.io`.
- Мы будем менять конфигурацию Gitlab, поэтому скачаем Chart.
```
helm fetch gitlab/gitlab-omnibus --version 0.1.37 --untar
cd gitlab-omnibus
```
- Поправьте gitlab-omnibus/values.yaml
```
baseDomain: example.com
legoEmail: you@example.com
```
- Поправить gitlab-omnibus/templates/gitlab/gitlab-svc.yaml 
```
cat << EOF > templates/gitlab/gitlab-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ template "fullname" . }}
  labels:
    app: {{ template "fullname" . }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
spec:
  selector:
    name: {{ template "fullname" . }}
  ports:
    - name: ssh
      port: 22
      targetPort: ssh
    - name: mattermost
      port: 8065
      targetPort: mattermost
    - name: registry
      port: 8105
      targetPort: registry
    - name: workhorse
      port: 8005
      targetPort: workhorse
    - name: prometheus
      port: 9090
      targetPort: prometheus
    - name: web
      port: 80
      targetPort: workhorse
EOF
```
- Поправить в gitlab-omnibus/templates/gitlab-config.yaml
```
cat << EOF > templates/gitlab-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ template "fullname" . }}-config
  labels:
    app: {{ template "fullname" . }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
data:
  external_scheme: http
  external_hostname: {{ template "fullname" . }}
  registry_external_scheme: https
  registry_external_hostname: registry.{{ .Values.baseDomain }}
  mattermost_external_scheme: https
  mattermost_external_hostname: mattermost.{{ .Values.baseDomain }}
  mattermost_app_uid: {{ .Values.mattermostAppUID }}
  postgres_user: gitlab
  postgres_db: gitlab_production
---
apiVersion: v1
kind: Secret
metadata:
  name: {{ template "fullname" . }}-secrets
  labels:
    app: {{ template "fullname" . }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
data:
  postgres_password: {{ .Values.postgresPassword }}
  initial_shared_runners_registration_token: {{ default "" .Values.initialSharedRunnersRegistrationToken | b64enc | quote }}
  mattermost_app_secret: {{ .Values.mattermostAppSecret | b64enc | quote }}
{{- if .Values.gitlabEELicense }}
  gitlab_ee_license: {{ .Values.gitlabEELicense | b64enc | quote }}
{{- end }}
EOF
```
- Поправить в `gitlab-omnibus/templates/ingress/gitlab-ingress.yaml`
```
cat << EOF > templates/ingress/gitlab-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: {{ template "fullname" . }}
  labels:
    app: {{ template "fullname" . }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
  annotations:
    kubernetes.io/tls-acme: "true"
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - gitlab.{{ .Values.baseDomain }}
    - registry.{{ .Values.baseDomain }}
    - mattermost.{{ .Values.baseDomain }}
    - prometheus.{{ .Values.baseDomain }}
    secretName: gitlab-tls
  rules:
  - host: {{ template "fullname" . }}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{ template "fullname" . }}
          servicePort: 8005
  - host: registry.{{ .Values.baseDomain }}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{ template "fullname" . }}
          servicePort: 8105
  - host: mattermost.{{ .Values.baseDomain }}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{ template "fullname" . }}
          servicePort: 8065
  - host: prometheus.{{ .Values.baseDomain }}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{ template "fullname" . }}
          servicePort: 9090
---
EOF
```
- Установим Gitlab `helm install --name gitlab . -f values.yaml`.
- Должно пройти несколько минут. Найдите выданный IP-адрес ingress-контроллера nginx. `kubectl get service -n nginx-ingress nginx`
```
NAME    TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                   AGE
nginx   LoadBalancer   10.11.253.79   35.238.203.247   80:31428/TCP,443:31374/TCP,22:30531/TCP   2m53s
```
- Поместите запись в локальный файл /etc/hosts (поставьте свой IP-адрес):
```
sudo sh -c 'echo "35.238.203.247 gitlab-gitlab staging production" >> /etc/hosts'
```
- 

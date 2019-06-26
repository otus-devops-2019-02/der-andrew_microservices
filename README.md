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

## The Hard Way
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
- Provisioning a CA and Generating TLS Certificates.
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
- 

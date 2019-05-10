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
- It works!!!
http://35.240.103.79:9292/
- Make multistage build post-py and run app.

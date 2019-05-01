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
- прощлый репозиторий мне так и не удалось подключить к трэвису. в этот раз, кроме строчки из ДЗ `travis encrypt "devops-team-otus:**********" --add notifications.slack.rooms --com` я подключил строчку, предложенную самим трэвисом `travis encrypt "devops-team-otus:**********" --add notifications.slack --com` и.... О чудо! Трэвис заговорил! Что явилось серебряной пулей сказать трудно. Поэкспериментируй!

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
- 

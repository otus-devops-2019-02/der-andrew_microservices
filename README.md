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

FROM ruby:2.2
RUN printf "deb http://archive.debian.org/debian/ jessie main\n\
deb-src http://archive.debian.org/debian/ jessie main\n\
deb http://security.debian.org jessie/updates main\n\
deb-src http://security.debian.org jessie/updates main" > /etc/apt/sources.list
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

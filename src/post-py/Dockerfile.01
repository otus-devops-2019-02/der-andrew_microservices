FROM python:3.6.0-alpine AS build

WORKDIR /app
ADD . /app
RUN apk add --no-cache build-base gcc
RUN pip install -r /app/requirements.txt

#FROM frolvlad/alpine-python3
FROM python:3.6.0-alpine

COPY --from=build /usr/local/lib/python3.6 /usr/local/lib/python3.6
COPY --from=build \
/usr/local/bin/chardetect \
/usr/local/bin/flask \
/usr/local/bin/

WORKDIR /app
ADD . /app

ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

ENTRYPOINT ["python3", "post_app.py"]

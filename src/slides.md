---
theme: uncover
class:
  - lead
  - invert
size: 16:9
footer: "Peter Fisher BSc MBCS howtocodewell.net @pfwd @howToCodeWell"
---

# How to reduce your Docker image size with Docker multi stage builds

---

# $ whoami = Peter Fisher
- Freelance Full Stack Web Developer
- Host of the How To Code Well
- Podcast howtocodewell.fm
- YouTube channel youtube.com/howtocodewell
- Twitch live coders team howtocodewell.net/live
- Discord server howtocodewell.net/discord
- Tutorials and courses howtocodewell.net

---

# How many of you use Docker?

---

# How many of you have use Docker multi stage builds?

---

# What are Docker multi stages

- Splits a Docker file into separate stages.
- A stage can be extended from a previous stage or image.
- Build artifacts can be copied from previous stages.

---

# Traditional Docker files

- Webserver (PHP Extensions, Nginx)
- Composer dependencies
- Frontend dependencies
- Configuration (Vhost, Permissions)

---

# Stage 1: The base layer

```
FROM php:7.4-apache-buster as webserver-base

LABEL maintainer="Peter Fisher"

ARG DEBIAN_FRONTEND=noninteractive
ENV APACHE_DOCUMENT_ROOT="/var/www/html/public"
RUN apt-get update --fix-missing \
    && apt-get install -y --no-install-recommends mariadb-client zlib1g-dev libcurl3-dev libssl-dev \
    && docker-php-ext-install pdo pdo_mysql iconv \
    && a2enmod rewrite \
    && a2enmod headers
```
---

# Stage 2: Composer

```
FROM composer as pm-backend

COPY composer.json composer.json
COPY composer.lock composer.lock

RUN composer install \
    --ignore-platform-reqs \
    --no-interaction \
    --no-plugins \
    --no-scripts \
    --prefer-dist
```

---

# Stage 3: The frontend packages

```
FROM node as pm-frontend

RUN mkdir -p /app/public

COPY package.json /app/

WORKDIR /app

RUN npm install
```

---

# Stage 4: Putting it all together

```
FROM webserver-base as webserver

COPY . /var/www/html
COPY --from=pm-backend /app/vendor/ /var/www/html/vendor/
COPY --from=pm-frontend /app/public/js/ /var/www/html/public/js/
COPY --from=pm-frontend /app/public/css/ /var/www/html/public/css/
```

---

# Advantages

- Build artifacts such as the vendor and node_modules can be copied from one stage to another
- Stages can be extended from other stages or images
- A constant build pipeline is created
- The final image size is far smaller as the previous stages are not combined with the final image.
- The attack surface is reduced as the build tools are not included in the final image

---

# Disadvantages
- Multi stage builds create complex Docker files.
- More post build scripts are needed to update dependencies
- A change to one stage may have unexpected side effects to a future stage
- You cannot run the last stage before the first stage
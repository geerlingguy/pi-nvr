FROM arm64v8/node:12.18.3-buster-slim

ENV DB_USER=majesticflame \
    DB_PASSWORD='' \
    DB_HOST='localhost' \
    DB_DATABASE=ccio \
    DB_PORT=3306 \
    SUBSCRIPTION_ID=sub_XXXXXXXXXXXX \
    PLUGIN_KEYS='{}' \
    SSL_ENABLED='false' \
    SSL_COUNTRY='CA' \
    SSL_STATE='BC' \
    SSL_LOCATION='Vancouver' \
    SSL_ORGANIZATION='Shinobi Systems' \
    SSL_ORGANIZATION_UNIT='IT Department' \
    SSL_COMMON_NAME='nvr.ninja' \
    DB_DISABLE_INCLUDED=false
ARG DEBIAN_FRONTEND=noninteractive

RUN mkdir -p /home/Shinobi /config /var/lib/mysql

RUN apt update -y
RUN apt install wget curl net-tools -y

# Install MariaDB server... the debian way
RUN if [ "$DB_DISABLE_INCLUDED" = "false" ] ; then set -ex; \
  { \
    echo "mariadb-server" mysql-server/root_password password '${DB_ROOT_PASSWORD}'; \
    echo "mariadb-server" mysql-server/root_password_again password '${DB_ROOT_PASSWORD}'; \
  } | debconf-set-selections; \
  apt-get update; \
  apt-get install -y \
    "mariadb-server" \
        socat \
  ; \
    find /etc/mysql/ -name '*.cnf' -print0 \
    | xargs -0 grep -lZE '^(bind-address|log)' \
    | xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/'; fi

RUN if [ "$DB_DISABLE_INCLUDED" = "false" ] ; then sed -ie "s/^bind-address\s*=\s*127\.0\.0\.1$/#bind-address = 0.0.0.0/" /etc/mysql/my.cnf; fi

# Install FFmpeg

RUN apt install -y software-properties-common \
        libfreetype6-dev \
        libgnutls28-dev \
        libmp3lame-dev \
        libass-dev \
        libogg-dev \
        libtheora-dev \
        libvorbis-dev \
        libvpx-dev \
        libwebp-dev \
        libssh2-1-dev \
        libopus-dev \
        librtmp-dev \
        libx264-dev \
        libx265-dev \
        yasm && \
    apt install -y \
        build-essential \
        bzip2 \
        coreutils \
        gnutls-bin \
        nasm \
        tar \
        x264

RUN apt install -y zip

RUN apt install -y \
                ffmpeg \
                git \
                make \
                g++ \
                gcc \
                pkg-config \
                python3 \
                wget \
                tar \
                sudo \
                xz-utils


WORKDIR /home/Shinobi
RUN git clone https://gitlab.com/Shinobi-Systems/Shinobi.git .
# COPY . .
#RUN rm -rf /home/Shinobi/plugins
# COPY ./plugins  /home/Shinobi/plugins
RUN chmod -R 777 /home/Shinobi/plugins
RUN npm i npm@latest -g && \
    npm install pm2 -g && \
    npm install --unsafe-perm
RUN cp Docker/pm2.yml ./

# Copy default configuration files
# COPY ./config/conf.json ./config/super.json /home/Shinobi/
RUN chmod -f +x /home/Shinobi/Docker/init.sh

VOLUME ["/home/Shinobi/videos"]
VOLUME ["/home/Shinobi/plugins"]
VOLUME ["/home/Shinobi/libs/customAutoLoad"]
VOLUME ["/config"]
VOLUME ["/var/lib/mysql"]

EXPOSE 8080

ENTRYPOINT ["/home/Shinobi/Docker/init.sh"]

CMD [ "pm2-docker", "pm2.yml" ]
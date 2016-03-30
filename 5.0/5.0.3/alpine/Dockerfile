FROM python:2.7-alpine
MAINTAINER "Plone Community" http://community.plone.org

RUN addgroup -g 500 plone \
 && adduser -S -D -G plone -u 500 plone \
 && mkdir -p /plone /data/filestorage /data/blobstorage \
 && chown -R plone:plone /plone /data

ENV PLONE_MAJOR=5.0
ENV PLONE_VERSION=5.0.3
ENV PLONE_MD5=6c0f560c3c5f23d8838c2df9a3b380f9

RUN apk add --no-cache --virtual .build-deps \
    curl \
    bzip2-dev \
    gcc \
    libc-dev \
    ncurses-dev \
    openssl-dev \
    readline-dev \
    zlib-dev \
    sudo \
    make \
    libxml2-dev \
    libxslt-dev \
    libjpeg-turbo-dev \
    libpng-dev openssl \
 && curl -SL -o Plone.tgz https://launchpad.net/plone/$PLONE_MAJOR/$PLONE_VERSION/+download/Plone-$PLONE_VERSION-UnifiedInstaller.tgz \
 && tar -zxvf Plone.tgz \
 && ./Plone-$PLONE_VERSION-UnifiedInstaller/install.sh \
      --password=admin \
      --daemon-user=plone \
      --owner=plone \
      --group=plone \
      --target=/plone \
      --instance=instance \
      --var=/data \
      standalone \
 && chown -R plone:plone /plone /data \
# && apk del .build-deps \
 && apk add --no-cache --virtual .run-deps \
    bash \
    libxml2 \
    libxslt \
    libjpeg-turbo \
 && rm -rf Plone* \
 && rm -rf /plone/buildout-cache/downloads/* \
 && find /plone \( -type f -a -name '*.pyc' -o -name '*.pyo' \) -exec rm -rf '{}' +

VOLUME /data/filestorage /data/blobstorage

COPY docker-initialize.py docker-entrypoint.sh /

EXPOSE 8080
USER plone
WORKDIR /plone/instance

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["start"]
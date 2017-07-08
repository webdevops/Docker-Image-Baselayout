# Docker Image Baselayout

Adds some convenience scripts to docker images


| File                                | Description                                                                               |
|:------------------------------------|:------------------------------------------------------------------------------------------|
| /usr/local/bin/apk-install          | `apk add` wrapper for Alpine                                                              |
| /usr/local/bin/apk-upgrade          | Upgrades all packages on Alpine Linux                                                     |
| /usr/local/bin/apt-add-repository   | Wrapper for `apt-add-repository` with auto cleanup                                        |
| /usr/local/bin/apt-install          | `apt-get install` wrapper for Debian/Ubuntu with auto cleanup                             |
| /usr/local/bin/apt-update           | `apt-get update` wrapper for Debian/Ubuntu for multiple apt-install runs                  |
| /usr/local/bin/apt-upgrade          | `apt-get dist-upgrade` wrapper for Debian/Ubuntu with auto cleanup                        |
| /usr/local/bin/yum-install          | `yum install` wrapper for RedHat with auto cleanup                                        |
| /usr/local/bin/yum-upgrade          | `yum upgrade` wrapper for RedHat with auto cleanup                                        |
| /usr/local/bin/docker-image-cleanup | Cleanup for docker images after package installations                                     |
| /usr/local/bin/docker-image-info    | Gets information about the current docker images (run `generate-dockerimage-info before`) |
| /usr/local/bin/generate-locales     | Generates system locales (eg. for number or date formatting)                              |


## Usage

### General docker images
needs wget installed

    RUN wget -O /tmp/baselayout-install.sh https://raw.githubusercontent.com/webdevops/Docker-Image-Baselayout/master/install.sh \
        && sh /tmp/baselayout-install.sh \
        && rm -f /tmp/baselayout-install.sh

### Multi stage docker build (requires docker 17.06 or later)

    FROM alpine:latest AS baselayout
    RUN apk add --no-cache ca-certificates wget \
        && update-ca-certificates \
        && wget -O /tmp/baselayout-install.sh https://raw.githubusercontent.com/webdevops/Docker-Image-Baselayout/master/install.sh \
        && sh /tmp/baselayout-install.sh /baselayout
        
    FROM ....
    COPY --from=baselayout /baselayout /

### Debian/Ubuntu docker images

    RUN apt-get update \
        && apt-get install -y -f --no-install-recommends wget ca-certificates \
        && wget -O /tmp/baselayout-install.sh https://raw.githubusercontent.com/webdevops/Docker-Image-Baselayout/master/install.sh?123 \
        && sh /tmp/baselayout-install.sh \
        && rm -f /tmp/baselayout-install.sh \
        && apt-get remove --auto-remove -y -f wget ca-certificates \
        && rm -rf /var/lib/apt/lists/* \
        && apt-get clean -y

### RedHat/Centos docker images

    RUN yum -y install wget \
        && wget -O /tmp/baselayout-install.sh https://raw.githubusercontent.com/webdevops/Docker-Image-Baselayout/master/install.sh \
        && sh /tmp/baselayout-install.sh \
        && rm -f /tmp/baselayout-install.sh \
        && yum -y erase wget \
        && yum clean all


### Alpine

    RUN apk add --no-cache ca-certificates wget --virtual .webdevops-baselayout-deps \
        && update-ca-certificates \
        && wget -O /tmp/baselayout-install.sh https://raw.githubusercontent.com/webdevops/Docker-Image-Baselayout/master/install.sh \
        && sh /tmp/baselayout-install.sh \
        && rm -f /tmp/baselayout-install.sh \
        && apk del .webdevops-baselayout-deps
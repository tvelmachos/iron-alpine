FROM alpine:3.18.3
# LABEL org.opencontainers.image.source="https://github.com/ironpeakservices/iron-alpine"

# make a pipe fail on the first failure
SHELL ["/bin/sh", "-o", "pipefail", "-c"]

# ensure we only use apk repositories over HTTPS (altough APK contain an embedded signature)
RUN echo "https://alpine.global.ssl.fastly.net/alpine/v$(cut -d . -f 1,2 < /etc/alpine-release)/main" > /etc/apk/repositories \
	&& echo "https://alpine.global.ssl.fastly.net/alpine/v$(cut -d . -f 1,2 < /etc/alpine-release)/community" >> /etc/apk/repositories

# The user the app should run as
ENV APP_USER=app
# The home directory
ENV APP_DIR="/$APP_USER"

RUN apk add --no-cache ca-certificates openssh

COPY entrypoint.sh /app/entrypoint.sh

# Add custom user and setup home directory
RUN adduser -s /bin/true -u 1001 -D -h $APP_DIR $APP_USER \
  && chown -R "$APP_USER" "$APP_DIR"\
  && chmod -R 700 "$APP_DIR" 

# Remove all but a handful of admin commands.
RUN find /sbin /usr/sbin \
  ! -type d -a ! -name apk -a ! -name ln  ! -name sshd \
  -delete

# Remove world-writeable permissions except for /tmp/
RUN find / -xdev -type d -perm +0002 -exec chmod o-w {} + \
	&& find / -xdev -type f -perm +0002 -exec chmod o-w {} + \
	&& chmod 777 /tmp/ \
  && chown $APP_USER:root /tmp/

# Remove unnecessary accounts, excluding current app user and root
RUN sed -i -r "/^($APP_USER|root|nobody)/!d" /etc/group \
  && sed -i -r "/^($APP_USER|root|nobody|ssh)/!d" /etc/passwd \
  && sed -i -r 's#^(.*):[^:]*$#\1:/sbin/nologin#' /etc/passwd \ 
  && while IFS=: read -r username _; do passwd -l "$username"; done < /etc/passwd || true

# Remove dangerous commands
RUN find /bin /etc /lib /sbin /usr -xdev \( \
  -iname hexdump -o \
  -iname chgrp -o \
  -iname ln -o \
  -iname od -o \
  -iname strings -o \
  -iname sudo \
  \) -delete

# Remove existing files/dirs, if any.
RUN rm -fr /var/spool/cron \
	&& rm -fr /etc/crontabs \
	&& rm -fr /etc/periodic \
  && rm -fr /etc/init.d /lib/rc /etc/conf.d /etc/inittab /etc/logrotate.d \
  && rm -fr /etc/sysctl* /etc/modprobe.d /etc/modules /etc/mdev.conf /etc/acpi \
  && rm -fr /root /etc/fstab \
  && find /bin /etc /lib /sbin /usr -xdev -type l -exec test ! -e {} \; -delete

# Ensure system dirs are owned by root and not writable by anybody else.
RUN find /bin /etc /lib /sbin /usr -xdev -type d \
  -exec chown root:root {} \; \
  -exec chmod 0755 {} \;

# default directory 

WORKDIR $APP_USER 
CMD  [ "/app/entrypoint.sh" ]
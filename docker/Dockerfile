# docker build 在根路径下，注意copy路径问题
FROM harbor.qyvideo.net/duanyifei-test/nginx:latest
MAINTAINER duanyifei@kingsoft.com

COPY /dist/* /usr/share/nginx/html/
COPY /docker/toolbox.conf /etc/nginx/conf.d
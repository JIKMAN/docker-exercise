FROM ubuntu:18.04
LABEL maintainer="Jungik Kim <wjddlr0303@gmail.com>"

# install apache
RUN apt-get update \
    && apt-get install -y apache2

# HTML 문서 생성
RUN echo "TEST WEB" > /var/www/html/index.html

# 포트 지정
EXPOSE 80

# 웹서버 동작 지정
CMD ["/usr/sbin/apache2ctl", "-DFOREGROUND"]

# docker build -t alpine:3.22.1 .
# 个人 镜像
# 3.22 版本
FROM alpine:3.22  

USER root

 
RUN apk --no-cache add \
    ca-certificates \
    tzdata \
    curl \
    busybox-extras \
    openssl \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone
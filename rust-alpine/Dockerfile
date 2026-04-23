# docker build -t rust-alpine:1.1 .
# https://github.com/sinlov/docker-rust-buster/blob/50f820e00f613ec7a437df48463aae0b72f5d42a/build.dockerfile#L21

FROM rust:1.95.0-alpine3.22

USER root

RUN apk --no-cache add \
    ca-certificates \
    tzdata \
    curl \
    busybox-extras \
    openssl \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone

ARG RUSTUP_DIST_SERVER=https://rsproxy.cn
ARG RUSTUP_UPDATE_ROOT=https://rsproxy.cn/rustup
ENV RUSTUP_DIST_SERVER=${RUSTUP_DIST_SERVER}
ENV RUSTUP_UPDATE_ROOT=${RUSTUP_UPDATE_ROOT}
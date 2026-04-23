# docker build -t alpine:1.0 .

FROM crpi-p35inr9lrgirg3t0.cn-hangzhou.personal.cr.aliyuncs.com/yaobo-box/ubuntu:20.04 AS cross-base

# 1. 合并更新+安装，清理缓存，减少镜像层
RUN apt-get update && \
    apt-get install --assume-yes --no-install-recommends \
    autoconf \
    automake \
    binutils \
    ca-certificates \
    curl \
    file \
    gcc \
    git \
    sudo \
    wget \
    libtool \
    m4 \
    g++ \
    gfortran \
    libc6-dev \
    apt-utils \
    libclang-dev \
    libevdev-dev \
    libssl-dev \
    make \
    pkg-config && \
    # 清理缓存，减小镜像体积
    rm -rf /var/lib/apt/lists/*

# 2. 复制 protoc 并授权（路径统一）
COPY ./protoc/protoc /usr/bin/protoc
RUN chmod +x /usr/bin/protoc

# 3. 设置 rustup 代理（ENV 永久生效）
ENV RUSTUP_DIST_SERVER=https://rsproxy.cn
ENV RUSTUP_UPDATE_ROOT=https://rsproxy.cn/rustup

# 4. 安装 Rust
RUN curl https://sh.rustup.rs -sSf > /tmp/rustup-init.sh && \
    chmod +x /tmp/rustup-init.sh && \
    sh /tmp/rustup-init.sh -y && \
    rm -rf /tmp/rustup-init.sh

# 5. 正确配置 PATH
ENV PATH="/root/.cargo/bin:$PATH"

# 6. 验证安装（只保留必要验证）
RUN rustc --version && cargo --version
# =====================================================
# CI-safe QingLong Dockerfile
# =====================================================

########################
# Stage 1: Node builder
########################
FROM node:18-alpine AS node-builder

ENV PNPM_HOME=/pnpm
ENV PATH=$PNPM_HOME:$PATH

RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    git \
    bash

RUN corepack enable \
 && corepack prepare pnpm@8.3.1 --activate

WORKDIR /build

COPY package.json pnpm-lock.yaml .npmrc ./

RUN pnpm install --prod

########################
# Stage 2: Runtime
########################
FROM python:3.11-alpine

ARG QL_MAINTAINER=gongymz
ARG QL_BRANCH=develop
ARG QL_URL=https://github.com/${QL_MAINTAINER}/qinglong.git

ENV QL_DIR=/ql \
    LANG=C.UTF-8 \
    TZ=Asia/Shanghai \
    SHELL=/bin/bash

VOLUME /ql/data
EXPOSE 5700

# ---------- system deps ----------
RUN apk add --no-cache \
    bash \
    curl \
    wget \
    git \
    tzdata \
    openssl \
    nodejs \
    npm \
    jq \
    openssh \
    procps \
    netcat-openbsd \
    unzip \
    perl \
    ca-certificates \
 && ln -sf /usr/share/zoneinfo/$TZ /etc/localtime \
 && echo "$TZ" > /etc/timezone \
 && git config --global user.email "qinglong@users.noreply.github.com" \
 && git config --global user.name "qinglong" \
 && git config --global --add safe.directory '*' \
 && rm -rf /var/cache/apk/*

# ---------- clone qinglong (NO shallow clone) ----------
RUN git clone -b ${QL_BRANCH} ${QL_URL} ${QL_DIR} \
 && cd ${QL_DIR} \
 && cp -f .env.example .env \
 && chmod +x shell/*.sh docker/*.sh

# ---------- clone static ----------
RUN git clone -b ${QL_BRANCH} https://github.com/${QL_MAINTAINER}/qinglong-static.git /static \
 && mkdir -p ${QL_DIR}/static \
 && cp -rf /static/* ${QL_DIR}/static \
 && rm -rf /static

# ---------- python deps ----------
RUN pip3 install --no-cache-dir requests

# ---------- node modules ----------
COPY --from=node-builder /build/node_modules ${QL_DIR}/node_modules

WORKDIR ${QL_DIR}

HEALTHCHECK --interval=5s --timeout=2s --retries=20 \
  CMD curl -sf http://127.0.0.1:5700/api/health || exit 1

ENTRYPOINT ["./docker/docker-entrypoint.sh"]

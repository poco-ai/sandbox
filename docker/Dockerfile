FROM debian:12 AS base

ENV DEBIAN_FRONTEND=noninteractive

# Base OS tools (shared by full/lite)
RUN apt-get update && apt-get install -y \
    sudo \
    curl \
    wget \
    vim \
    git \
    ca-certificates \
    gnupg \
    software-properties-common \
    locales \
    && locale-gen zh_CN.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN:zh \
    LC_ALL=zh_CN.UTF-8

# Create user ubuntu and grant sudo privileges
RUN useradd -m -d /home/ubuntu -s /bin/bash ubuntu && \
    echo "ubuntu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ubuntu

# Install Python (Debian default, currently 3.11)
RUN apt-get update && \
    apt-get install -y python3 python3-venv python3-dev python3-pip python-is-python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Node.js 20.x
RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install -y nodejs && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install global npm packages
RUN npm install -g typescript ts-node pnpm yarn

# Create directories and set ownership
RUN mkdir -p /app /workspace && chown -R ubuntu:ubuntu /app /workspace

ENV PATH="/home/ubuntu/.local/bin:${PATH}"
WORKDIR /app

# -----------------------------------------------------------------------------
# Lite image (no desktop/browser)
# -----------------------------------------------------------------------------
FROM base AS lite

USER ubuntu
RUN mkdir -p /home/ubuntu/.npm-global
ENV NPM_CONFIG_PREFIX=/home/ubuntu/.npm-global
ENV PATH="/home/ubuntu/.npm-global/bin:${PATH}"
# Initialize npm global directory structure by installing a minimal package
RUN npm install -g --no-save npm@latest
COPY --chown=ubuntu:ubuntu requirements.txt .
RUN pip3 install --user --no-cache-dir -r requirements.txt

VOLUME /workspace

CMD ["sleep", "infinity"]

# -----------------------------------------------------------------------------
# Full image (desktop + browser)
# -----------------------------------------------------------------------------
FROM base AS full

# Services used by the full sandbox image
RUN apt-get update && apt-get install -y \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

# Install Caddy
RUN curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /etc/apt/keyrings/caddy-stable-archive-keyring.gpg \
    && curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list \
    && sed -i 's|/usr/share/keyrings/|/etc/apt/keyrings/|g' /etc/apt/sources.list.d/caddy-stable.list \
    && apt-get update \
    && apt-get install -y caddy \
    && rm -rf /var/lib/apt/lists/*

# Install code-server
RUN curl -fsSL https://code-server.dev/install.sh | sh

RUN mkdir -p /etc/supervisor/conf.d /etc/caddy

# Desktop / VNC / noVNC tooling
RUN apt-get update && apt-get install -y \
    xvfb \
    x11vnc \
    openbox \
    xterm \
    websockify \
    && rm -rf /var/lib/apt/lists/*

# Install noVNC from source (latest stable version)
RUN cd /tmp && \
    wget -qO- https://github.com/novnc/noVNC/archive/refs/tags/v1.6.0.tar.gz | tar xzf - && \
    mv noVNC-1.6.0 /usr/share/novnc && \
    rm -rf /tmp/noVNC-1.6.0.tar.gz

# Install Chromium browser (supports both amd64 and arm64)
RUN apt-get update && apt-get install -y \
    chromium \
    && rm -rf /var/lib/apt/lists/*

ENV DISPLAY=:1

# Chinese fonts and common multimedia tooling (used by some Python packages)
RUN apt-get update && apt-get install -y \
    fonts-noto-cjk \
    fonts-noto-color-emoji \
    ffmpeg \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY Caddyfile /etc/caddy/Caddyfile

USER ubuntu
RUN mkdir -p /home/ubuntu/.npm-global
ENV NPM_CONFIG_PREFIX=/home/ubuntu/.npm-global
ENV PATH="/home/ubuntu/.npm-global/bin:${PATH}"
# Initialize npm global directory structure by installing a minimal package
RUN npm install -g --no-save npm@latest
COPY --chown=ubuntu:ubuntu requirements.txt .
RUN pip3 install --user --no-cache-dir -r requirements.txt

# Install Playwright browser (installed to ~/.cache/ms-playwright)
RUN playwright install chromium --with-deps

USER root
VOLUME /workspace

EXPOSE 8080

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

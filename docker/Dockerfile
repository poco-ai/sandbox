FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Configure apt source to use Aliyun mirror
RUN sed -i 's|http://archive.ubuntu.com/ubuntu/|http://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list && \
    sed -i 's|http://security.ubuntu.com/ubuntu/|http://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list && \
    sed -i 's|http://ports.ubuntu.com/ubuntu-ports/|http://mirrors.aliyun.com/ubuntu-ports/|g' /etc/apt/sources.list

# Update and install basic tools
RUN apt-get update && apt-get install -y \
    sudo \
    curl \
    wget \
    vim \
    git \
    ca-certificates \
    gnupg \
    software-properties-common \
    xvfb \
    x11vnc \
    openbox \
    xterm \
    novnc \
    websockify \
    supervisor \
    apt-transport-https \
    && rm -rf /var/lib/apt/lists/*

# Create user ubuntu and grant sudo privileges
RUN useradd -m -d /home/ubuntu -s /bin/bash ubuntu && \
    echo "ubuntu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ubuntu

# Install Python 3.10 from deadsnakes PPA
RUN add-apt-repository ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y python3.10 python3.10-venv python3.10-dev python3-pip && \
    update-alternatives --install /usr/bin/python python /usr/bin/python3.10 1 && \
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1 && \
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

# Configure npm to use Aliyun mirror
RUN npm config set registry https://registry.npmmirror.com

# Install global npm packages
RUN npm install -g typescript ts-node pnpm yarn

# Install Caddy
RUN curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /etc/apt/keyrings/caddy-stable-archive-keyring.gpg \
    && curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list \
    && sed -i 's|/usr/share/keyrings/|/etc/apt/keyrings/|g' /etc/apt/sources.list.d/caddy-stable.list \
    && apt-get update \
    && apt-get install -y caddy \
    && rm -rf /var/lib/apt/lists/*

# Install Google Chrome
RUN wget -qO- https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor > /etc/apt/keyrings/google-chrome.gpg \
    && echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list \
    && apt-get update \
    && apt-get install -y google-chrome-stable \
    && rm -rf /var/lib/apt/lists/*

ENV DISPLAY=:1

# Install Chinese fonts and language support
RUN apt-get update && apt-get install -y \
    fonts-noto-cjk \
    fonts-noto-color-emoji \
    language-pack-zh-hans \
    locales \
    ffmpeg \
    && locale-gen zh_CN.UTF-8 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN:zh \
    LC_ALL=zh_CN.UTF-8

# Install code-server
RUN curl -fsSL https://code-server.dev/install.sh | sh

RUN mkdir -p /etc/supervisor/conf.d
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY Caddyfile /etc/caddy/Caddyfile

# Create directories and set ownership before switching user
RUN mkdir -p /app /workspace && chown -R ubuntu:ubuntu /app /workspace

# ============================================================================
# >>>  SWITCH TO UBUNTU USER  <<<
# >>>  All user-level configs (pip packages, caches, dotfiles) go here  <<<
# ============================================================================
USER ubuntu
WORKDIR /app

# Configure pip mirror for ubuntu user
RUN pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# Install Python dependencies (installed to ~/.local)
COPY --chown=ubuntu:ubuntu requirements.txt .
RUN pip3 install --user --no-cache-dir -r requirements.txt

# Add user local bin to PATH
ENV PATH="/home/ubuntu/.local/bin:${PATH}"

# Install Playwright browser (installed to ~/.cache/ms-playwright)
RUN playwright install chromium --with-deps

# ============================================================================
# >>>  END OF UBUNTU USER SETUP, SWITCH BACK TO ROOT  <<<
# ============================================================================
USER root

# Declare volumes (optional mounting, works without mounting too)
VOLUME /home/ubuntu
VOLUME /workspace

EXPOSE 8080

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

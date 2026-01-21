# Vibe Coding Terminal - code-server with GitHub Copilot
FROM codercom/code-server:latest

USER root

# Install base dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    wget \
    unzip \
    gnupg \
    lsb-release \
    software-properties-common \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js (LTS)
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Install Python 3 with pip
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && rm -rf /var/lib/apt/lists/*

# Install Go
RUN wget -q https://go.dev/dl/go1.22.0.linux-amd64.tar.gz \
    && tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz \
    && rm go1.22.0.linux-amd64.tar.gz

# Install Docker CLI
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
    && apt-get update && apt-get install -y docker-ce-cli \
    && rm -rf /var/lib/apt/lists/*

# Install kubectl
RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" > /etc/apt/sources.list.d/kubernetes.list \
    && apt-get update && apt-get install -y kubectl \
    && rm -rf /var/lib/apt/lists/*

# Install Terraform
RUN wget -q https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip \
    && unzip terraform_1.7.0_linux_amd64.zip -d /usr/local/bin \
    && rm terraform_1.7.0_linux_amd64.zip

# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
    && unzip awscliv2.zip \
    && ./aws/install \
    && rm -rf awscliv2.zip aws

# Set up environment for coder user
USER coder

# Install Rust for coder user
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

# Set PATH for all tools
ENV PATH="/home/coder/.cargo/bin:/usr/local/go/bin:/usr/local/bin:${PATH}"
ENV GOPATH="/home/coder/go"

# Configure code-server to use Microsoft VS Code marketplace for Copilot extensions
ENV EXTENSIONS_GALLERY='{"serviceUrl":"https://marketplace.visualstudio.com/_apis/public/gallery","itemUrl":"https://marketplace.visualstudio.com/items","cacheUrl":"https://vscode.blob.core.windows.net/gallery/index","controlUrl":""}'

# Install GitHub Copilot extension
RUN code-server --install-extension GitHub.copilot

# Install GitHub Copilot Chat extension
RUN code-server --install-extension GitHub.copilot-chat

# Configure optimal vibe-coding settings
RUN mkdir -p /home/coder/.local/share/code-server/User

# Create default settings for optimal coding experience
RUN echo '{\n\
  "editor.fontSize": 14,\n\
  "editor.fontFamily": "\"Fira Code\", Consolas, monospace",\n\
  "editor.fontLigatures": true,\n\
  "editor.tabSize": 2,\n\
  "editor.formatOnSave": true,\n\
  "editor.minimap.enabled": true,\n\
  "editor.wordWrap": "on",\n\
  "editor.bracketPairColorization.enabled": true,\n\
  "terminal.integrated.fontSize": 13,\n\
  "workbench.colorTheme": "Default Dark Modern",\n\
  "files.autoSave": "afterDelay",\n\
  "files.autoSaveDelay": 1000,\n\
  "github.copilot.enable": {\n\
    "*": true\n\
  },\n\
  "github.copilot.editor.enableAutoCompletions": true\n\
}' > /home/coder/.local/share/code-server/User/settings.json

# Set working directory
WORKDIR /home/coder/project

# Expose code-server port
EXPOSE 8080

# Default command (can be overridden in docker-compose)
CMD ["code-server", "--bind-addr", "0.0.0.0:8080", "--auth", "none", "/home/coder/project"]

# Vibe Coding Terminal - code-server with GitHub Copilot
FROM codercom/code-server:latest

USER root

# Install dependencies for extensions
RUN apt-get update && apt-get install -y \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

USER coder

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

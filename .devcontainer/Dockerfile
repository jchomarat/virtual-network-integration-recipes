# Find the Dockerfile for mcr.microsoft.com/azure-functions/python:4-python3.8-core-tools at this URL
# https://github.com/Azure/azure-functions-docker/blob/dev/host/4/bullseye/amd64/python/python38/python38-core-tools.Dockerfile
FROM mcr.microsoft.com/devcontainers/base:bullseye

# Args
# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# [Choice] Node.js version: none, lts, 16, 14, 12, 10
ARG NODE_VERSION="14"

# Install system tools
RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends apt-utils zip unzip curl bash-completion dnsutils figlet \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

RUN wget https://dot.net/v1/dotnet-install.sh && chmod +x dotnet-install.sh && ./dotnet-install.sh --version latest

ENV DOTNET_ROOT=$HOME/.dotnet
ENV PATH=$PATH:$HOME/.dotnet:$HOME/.dotnet/tools

# Install Azure Function Tools here for amd64 architecture as support for arm64/aarch64 in Linux
# is not supported it yet (https://github.com/Azure/azure-functions-core-tools/issues/3112)
# If, on Apple Silicon for instance you need Azure Functions Tools, you will need to build
# the container in amd64 and run it emuulated by running
# export DOCKER_DEFAULT_PLATFORM="linux/amd64" && code .
ARG TARGETPLATFORM
RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list && curl -sL https://packages.microsoft.com/keys/microsoft.asc | (OUT=$(apt-key add - 2>&1) || echo $OUT) && apt-get update && apt-get install -y azure-functions-core-tools-4; fi

# Save command line history 
RUN echo "export HISTFILE=/home/$USERNAME/commandhistory/.bash_history" >> "/home/$USERNAME/.bashrc" \
    && echo "export PROMPT_COMMAND='history -a'" >> "/home/$USERNAME/.bashrc" \
    && mkdir -p /home/$USERNAME/commandhistory \
    && touch /home/$USERNAME/commandhistory/.bash_history \
    && chown -R $USERNAME /home/$USERNAME/commandhistory

# Git command prompt and bash completion
RUN git clone https://github.com/magicmonty/bash-git-prompt.git /home/$USERNAME/.bash-git-prompt --depth=1 \
    && echo "if [ -f \"/home/$USERNAME/.bash-git-prompt/gitprompt.sh\" ]; then GIT_PROMPT_ONLY_IN_REPO=1 && source /home/$USERNAME/.bash-git-prompt/gitprompt.sh; fi" >> "/home/$USERNAME/.bashrc" \
    && echo "source /usr/share/bash-completion/bash_completion" >> "/home/$USERNAME/.bashrc"

ENV GIT_PROMPT_START='\033[1;36mvnet-recipes>\033[0m\033[0;33m\w\a\033[0m'

# Fetch the latest Bicep CLI binary.  This is used for running Bicep files from Azure PowerShell.
# As of July 2021, Azure PowerShell does not have the capability to install the Bicep CLI.
## https://github.com/Azure/bicep/issues/3447
# RUN mkdir -p /tmp/bicep-downloads \
#     && curl -Lo /tmp/bicep-downloads/bicep.bin https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64 \
#     # Mark it as executable
#     && chmod +x /tmp/bicep-downloads/bicep.bin \
#     # Add bicep to your PATH (requires admin)
#     && mv /tmp/bicep-downloads/bicep.bin /usr/local/bin/bicep

# Uncomment if not running as root (e.g., vscode).
USER $USERNAME
# [Choice] Python version: 3, 3.8, 3.7, 3.6
ARG VARIANT=3.8
FROM mcr.microsoft.com/vscode/devcontainers/python:${VARIANT}

ARG TERRAFORM_VERSION="0.14.7"
ARG POETRY_VERSION="1.1.4"

# [Option] Install Node.js
ARG INSTALL_NODE="true"
ARG NODE_VERSION="lts/*"
RUN if [ "${INSTALL_NODE}" = "true" ]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

# [Optional] If your pip requirements rarely change, uncomment this section to add them to the image.
# COPY requirements.txt /tmp/pip-tmp/
# RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt \
#    && rm -rf /tmp/pip-tmp

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment this line to install global node packages.
# RUN su vscode -c "source /usr/local/share/nvm/nvm.sh && npm install -g <your-package-here>" 2>&1

# Base python dependencies
RUN pip3 --disable-pip-version-check --no-cache-dir install poetry==${POETRY_VERSION} \
   && rm -rf /tmp/pip-tmp

# Install terraform
RUN apt-get update \
   && apt-get install unzip \
   && curl -O -L https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
   && unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
   && chown vscode terraform \
   && mv terraform /usr/local/bin \
   && rm terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
   # Clean up
   && apt-get autoremove -y \
   && apt-get clean -y \
   && rm -rf /var/lib/apt/lists/*

#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

# Azure ML does not support Debian 10 yet, so cannot use Anaconda image as base
FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-18.04

ARG ANACONDA_VERSION=2020.02

# Same terraform version as azure cloud cli
ARG TERRAFORM_VERSION="0.12.23"

ARG AZURE_ML_SDK_EXTRAS=notebooks

ARG DEFAULT_UTILS="\
    pylint \
    flake8 \
    autopep8 \
    black \
    pytest \
    yapf \
    mypy \
    pydocstyle \
    pycodestyle \
    bandit"

# The javascript-node image includes a non-root node user with sudo access. Use 
# the "remoteUser" property in devcontainer.json to use it. On Linux, the container 
# user's GID/UIDs will be updated to match your local UID/GID when using the image
# or dockerFile property. Update USER_UID/USER_GID below if you are using the
# dockerComposeFile property or want the image itself to start with different ID
# values. See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG CONDA_INSTALL_PATH=/opt/conda
ENV PATH=${CONDA_INSTALL_PATH}/bin:${PATH}
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    #
    # Alter vscode user as needed
    && if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then \
        groupmod --gid $USER_GID $USERNAME \
        && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
        && chown -R $USER_UID:$USER_GID /home/$USERNAME; \
    fi \
    #
    # Install Docker CLI
    && apt-get install -y gnupg-agent software-properties-common \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    #
    # Set up Anaconda - adapted for Ubuntu from https://github.com/ContinuumIO/docker-images/blob/master/anaconda3/debian/Dockerfile
    # Use vscode user for the installation so that it can be used to manage the conda environment.
    && apt-get install -y bzip2 libglib2.0-0 libxext6 libsm6 libxrender1 gcc g++ \
    && mkdir -p ${CONDA_INSTALL_PATH} \
    && chown ${USERNAME}:root /opt/conda \
    && echo "Downloading Anaconda..." \
    && su --login -c "wget -q https://repo.anaconda.com/archive/Anaconda3-${ANACONDA_VERSION}-Linux-x86_64.sh -O /tmp/anaconda-install.sh \
        && /bin/bash /tmp/anaconda-install.sh -u -b -p ${CONDA_INSTALL_PATH}" ${USERNAME} 2>&1  \
    && rm /tmp/anaconda-install.sh \
    && ln -s ${CONDA_INSTALL_PATH}/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
    # Add conda init to .bashrc/.zshrc, tweak ownership if UID was changed
    && export SNIPPET="export PATH=\$PATH:\$HOME/.local/bin \
        && if [ \"\$(stat -c '%U' ${CONDA_INSTALL_PATH})\" != \"${USERNAME}\" ]; then \
            sudo chown -R ${USERNAME}:root ${CONDA_INSTALL_PATH}; \
        fi \
        && . ${CONDA_INSTALL_PATH}/etc/profile.d/conda.sh \
        && conda activate base" \
    && echo "$SNIPPET" | tee -a /root/.bashrc >> /home/${USERNAME}/.bashrc \
    && echo "$SNIPPET" | tee -a /root/.zshrc >> /home/${USERNAME}/.zshrc \
    && find ${CONDA_INSTALL_PATH}/ -follow -type f -name '*.a' -delete \
    && find ${CONDA_INSTALL_PATH}/ -follow -type f -name '*.js.map' -delete \
    && ${CONDA_INSTALL_PATH}/bin/conda clean -afy \
    #
    # Install and Azure ML SDK as vscode user so it can be updated by both users
    && su --login -c "${CONDA_INSTALL_PATH}/bin/pip install --no-cache-dir --upgrade azureml-sdk[${AZURE_ML_SDK_EXTRAS}]" ${USERNAME} 2>&1 \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Install custom requirements
RUN apt-get update \
    #
    # Install azure cli with devops and ml extensions
    && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null \
    && AZ_REPO=$(lsb_release -cs) \
    && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | tee /etc/apt/sources.list.d/azure-cli.list \
    && apt-get update \
    && apt-get install azure-cli \
    #
    # Install azure ML and devops extensions
    && az extension add -n azure-cli-ml \
    && az extension add -n azure-devops \
    #
    # Install Ansible
    && apt-get install -y libssl-dev libffi-dev \
    && /opt/conda/bin/pip install ansible[azure] \
    #
    # Install azureml-dataprep pandas support and dev dependencies
    && su --login -c "${CONDA_INSTALL_PATH}/bin/pip install --no-cache-dir  azureml-dataprep[pandas] ${DEFAULT_UTILS}" ${USERNAME} 2>&1 \
    #
    # Install apt-get dev dependencies
    && apt-get install -y \
    vim \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

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

# Install KubeCtl and Helm
RUN curl -sSL -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
    && chmod +x /usr/local/bin/kubectl \
    && curl -s https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash -

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

# Find the Dockerfile for mcr.microsoft.com/azure-functions/python:3.0-python3.8-core-tools at this URL
# https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python38/python38-core-tools.Dockerfile
FROM mcr.microsoft.com/azure-functions/python:3.0-python3.8-core-tools

ARG TERRAFORM_VERSION="0.12.23"

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
    # Install apt-get dev dependencies
    && apt-get install -y \
    vim \
    git \
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
#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

# Find the Dockerfile for mcr.microsoft.com/azure-functions/python:3.0-python3.8-core-tools at this URL
# https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python38/python38-core-tools.Dockerfile
FROM mcr.microsoft.com/azure-functions/python:3.0-python3.8-core-tools

ARG TERRAFORM_VERSION="0.12.29"
ARG DEFAULT_UTILS="\
    pylint \
    flake8 \
    black \
    pytest"

# Install custom requirements
RUN apt-get update \
    #
    # Install apt-get dev dependencies
    && apt-get install -y \
    vim \
    git \
    #
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
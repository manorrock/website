FROM node:18

# Install basic development tools
RUN apt update && \
    apt install -y less man-db npm sudo 

# Ensure default `node` user has access to `sudo`
ARG USERNAME=node
RUN echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

USER node
RUN cd $HOME && \
    npm init --yes && \
    npm install honkit --save-dev && \
    npm install gitbook-plugin-hints --save-dev

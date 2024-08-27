FROM node:20-alpine3.18

WORKDIR /zenn/cli

ENV NPM_CONFIG_PREFIX /home/node/.npm-global
ENV PATH $PATH:/home/node/.npm-global/bin

RUN apk upgrade --no-cache && \
    apk add --update --no-cache vim git make curl bash


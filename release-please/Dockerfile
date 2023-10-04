FROM node:20-alpine

WORKDIR /home/node/app

ENV NODE_ENV=production

COPY package*.json ./

RUN npm ci

USER node

ENTRYPOINT ["npx", "release-please"]

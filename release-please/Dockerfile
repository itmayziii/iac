FROM node:20-alpine

WORKDIR /home/node/app

ENV NODE_ENV=production

RUN npm install -g release-please

USER node

ENTRYPOINT ["npx", "release-please"]

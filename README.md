# docker-node-chromium-alpine

A Docker image with preinstalled Chromium and Node.JS on Alpine Linux.
Good minimal base image for users of scraping libraries like
[Puppeteer](https://github.com/GoogleChrome/puppeteer/).

## Repository

https://github.com/shivjm/docker-node-chromium-alpine/

## Issues

https://github.com/shivjm/docker-node-chromium-alpine/issues/

## Tags

See all tags at [Docker Hub
(shivjm/node-chromium-alpine)](https://hub.docker.com/repository/docker/shivjm/node-chromium-alpine).

## Example Dockerfiles

Simple:

```Dockerfile
FROM shivjm/node-chromium-alpine

WORKDIR /usr/src/app

COPY package.json package-lock.json ./

RUN npm ci

COPY src .

ENTRYPOINT ["npm", "start"]
```

Multi-stage build to separate development dependencies from
production:

```Dockerfile
FROM node:13-alpine

WORKDIR /usr/src/app-deps

COPY package.json package-lock.json ./

RUN npm ci --quiet

COPY . .

RUN npm run --quiet compile-my-code && \
  npm prune --quiet --production

FROM shivjm/node-chromium-alpine:13

USER node

WORKDIR /usr/src/app

COPY package.json package-lock.json ./

COPY --from=build /usr/src/app-deps/node_modules ./node_modules

COPY --from=build /usr/src/app-deps/dist ./dist

ENV NODE_ENV=production

ENTRYPOINT ["npm", "start", "--quiet"]
```

## Puppeteer

When you install Puppeteer, it also downloads a known version of
Chromium to store under `node_modules`, and defaults to using that
binary. You can skip this download using the environment variable
`PUPPETEER_SKIP_CHROMIUM_DOWNLOAD`. You’ll also need to set
`PUPPETEER_EXECUTABLE_PATH` to the installed Chromium. A partial
example:

```Dockerfile
# (setup elided)

# install dependencies but not Chromium:

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1

RUN npm install

# make Puppeteer use correct binary:

ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser

# (other build details elided)
```

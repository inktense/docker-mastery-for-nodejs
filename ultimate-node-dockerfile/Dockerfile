FROM node:18-slim as base
# labels from https://github.com/opencontainers/image-spec/blob/master/annotations.md
LABEL org.opencontainers.image.authors=BB
LABEL org.opencontainers.image.title="Docker project"
LABEL org.opencontainers.image.licenses=MIT

ENV NODE_ENV=production

EXPOSE 80

WORKDIR /app

COPY package*.json ./

RUN apt-get update -qq && apt-get install -qy \ 
    tini 

RUN npm config list
# npm ci only uses the package-lock.json file. Cache clean keeps the image small, just how we like it for production. 
RUN npm ci \
    && npm cache clean --force

ENV PATH /app/node_modules/.bin:$PATH
ENTRYPOINT ["/usr/bin/tini", "--"]

CMD ["node", "server.js"]

####
# DEV stage
####
FROM base as dev
ENV NODE_ENV=development

# # NOTE: these apt dependencies are only needed
# # for testing. they shouldn't be in production
RUN apt-get update -qq \
    && apt-get install -qy --no-install-recommends \
    bzip2 \
    ca-certificates \
    curl \
    libfontconfig \
    && rm -rf /var/lib/apt/lists/*

RUN npm config list

#  Here we need to install dev dependencies that we don't have in prod 
RUN npm install --only=development \
    && npm cache clean --force

USER node

CMD ["nodemon", "server.js"]

####
# TEST stage
####
FROM dev as test

COPY . .

# RUN npm audit
# Scan images for vulnerabilities
COPY --from=aquasec/trivy:latest /usr/local/bin/trivy /usr/local/bin/trivy
USER root
# RUN trivy filesystem --exit-code 1 --no-progress /



####
# Intermediary stage
####
FROM test as preprod
RUN rm -rf ./tests && rm -rf ./node_modules


####
# PROD stage
####
FROM base as prod
COPY --from=preprod /app /app

HEALTHCHECK CMD curl http:127.0.0.1/ || exit 1
USER node
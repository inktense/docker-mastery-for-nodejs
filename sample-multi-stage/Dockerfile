FROM node:18-alpine as prod
ENV NODE_ENV=production
EXPOSE 3000
RUN apk add --no-cache tini
WORKDIR /app
COPY package*.json ./

RUN npm ci && npm cache clean --force 

COPY . .

ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "./bin/www"]

# Dockerfile for development
FROM prod as dev
ENV NODE_ENV=development
RUN npm install && npm cache clean --force

CMD ["node_modules/nodemon/bin/nodemon.js"  "./bin/www", "--inspect=0.0.0.0:9229"]

# Dockerfile for testing
FROM dev as test
ENV NODE_ENV=development
RUN npm install && npm cache clean --force

CMD ["npm", "run", "test"]

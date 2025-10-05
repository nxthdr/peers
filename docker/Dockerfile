FROM node:lts-alpine AS build

WORKDIR /app
COPY package.json ./

# Install dependencies - let npm resolve optional deps for the current platform
RUN npm install

COPY . .
RUN npm run build

FROM caddy:latest

WORKDIR /www/html
COPY --from=build /app/dist .
COPY ./docker/config/Caddyfile /etc/caddy/Caddyfile

EXPOSE 80/tcp
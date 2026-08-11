# --- STAGE 1: Build ---
FROM node:22-alpine AS build-env
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install --legacy-peer-deps

COPY . .
RUN npm run build

FROM nginx:1.29-alpine

RUN apk upgrade --no-cache

# Copy built assets
COPY --from=build-env /app/dist /usr/share/nginx/html

# Configure Nginx
RUN rm -f /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Move PID file to /tmp so Nginx can write to it without root access
RUN sed -i 's#pid.*#pid /tmp/nginx.pid;#' /etc/nginx/nginx.conf

# Generate env-config.js at container startup from the runtime VITE_* env vars.
# nginx's entrypoint runs every /docker-entrypoint.d/*.sh before starting nginx.
COPY env.sh /docker-entrypoint.d/40-env-config.sh
RUN chmod +x /docker-entrypoint.d/40-env-config.sh
# The startup script rewrites env-config.js in place, so the served dir must be
# writable by the non-root runtime user (USER 1001 below).
RUN chmod -R 777 /usr/share/nginx/html

RUN mkdir -p /tmp
RUN chmod -R 777 /tmp

RUN mkdir -p /var/cache/nginx/client_temp 
RUN mkdir -p /var/cache/nginx/proxy_temp 
RUN mkdir -p /var/cache/nginx/fastcgi_temp
RUN mkdir -p /var/cache/nginx/uwsgi_temp
RUN mkdir -p /var/cache/nginx/scgi_temp
RUN mkdir -p /var/run

USER 1001

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
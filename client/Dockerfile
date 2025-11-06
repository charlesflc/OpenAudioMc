# ---------- Build Vite/React ----------
FROM node:20-alpine AS builder
WORKDIR /app

# Install deps (cache-friendly)
COPY package*.json ./
RUN npm ci

# Copie du reste et build
COPY . .
ARG RUN_LINT=true
RUN if [ "$RUN_LINT" = "true" ]; then npm run lint; else echo "Skipping lint"; fi
RUN npm run build

# ---------- Runtime Nginx ----------
FROM nginx:alpine

# Conf Nginx: SPA fallback -> index.html
RUN rm -f /etc/nginx/conf.d/default.conf
COPY --link --chown=root:root <<'NGINXCONF' /etc/nginx/conf.d/default.conf
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;

    gzip on;
    gzip_types text/plain application/javascript application/json text/css text/xml image/svg+xml;
    gzip_min_length 1024;

    location / {
        try_files $uri $uri/ /index.html;
    }
    # Cache long pour assets fingerprintés
    location ~* \.(?:js|css|woff2?|ttf|eot|ico|svg|png|jpg|jpeg|gif)$ {
        expires 30d;
        access_log off;
        try_files $uri =404;
    }
}
NGINXCONF

# Vite produit "build" selon vite.config.js
COPY --from=builder /app/build/ /usr/share/nginx/html/

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://127.0.0.1/ >/dev/null || exit 1

EXPOSE 80

# syntax=docker/dockerfile:1
# Production Dockerfile for static SPA (Vite/React)
# --- Build stage ---
FROM node:20-bookworm-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm pkg delete scripts.prepare scripts.postinstall >/dev/null 2>&1 || true && unset NPM_CONFIG_OMIT && npm install --include=optional --no-audit --no-fund || npm install --include=optional --no-audit --no-fund --legacy-peer-deps
COPY public ./public
COPY src ./src
COPY vite.config.js ./vite.config.js
COPY index.html ./index.html
RUN npm run postinstall --if-present
RUN npm run build

# --- Runtime: nginx ---
FROM nginxinc/nginx-unprivileged:1.27-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -q --spider http://localhost:8080/healthz || exit 1
CMD ["nginx", "-g", "daemon off;"]

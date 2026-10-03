# ---- Build stage ----
FROM --platform=linux/amd64 node:22-slim AS build

WORKDIR /app

# Framework-prefixed vars bake into the bundle at build time.
# None are used in src/ today; declare the pattern so CI can pass them later.
ARG VITE_API_BASE_URL
ENV VITE_API_BASE_URL=${VITE_API_BASE_URL}

# Install full deps (vite is a devDependency, needed for the build).
# --include=optional ships per-platform native binaries (esbuild/rollup).
COPY package.json package-lock.json ./
RUN npm ci --include=optional

COPY . .
RUN npm run build

# ---- Runtime stage ----
FROM --platform=linux/amd64 node:22-slim

# Static file server, materialised at build time (not fetched at runtime).
RUN npm install -g serve@14 && npm cache clean --force

WORKDIR /app
COPY --from=build /app/dist ./dist

# Numeric UID required for Kubernetes runAsNonRoot.
USER 1000

# Pod spec may override PORT; default matches EXPOSE.
ENV PORT=8080
EXPOSE 8080

# Base image is slim (no curl/wget) — probe with node's built-in fetch.
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD node -e "fetch('http://127.0.0.1:'+(process.env.PORT||8080)+'/').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"

# sh -c + exec so ${PORT} expands and serve runs as PID 1.
CMD ["sh", "-c", "exec serve -s dist -l tcp://0.0.0.0:${PORT}"]

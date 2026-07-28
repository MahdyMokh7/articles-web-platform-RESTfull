# ---------------------------------------------------------------------------
# Stage 1: Build
# node:22-alpine has everything npm/Vite need to produce the static bundle.
# It's discarded afterwards, so its size doesn't affect the shipped image.
# ---------------------------------------------------------------------------
FROM node:22-alpine AS build
WORKDIR /app

# Install dependencies in their own cached layer before copying source.
COPY package*.json ./
RUN npm ci

COPY . .

# Vite inlines VITE_* variables into the bundle at build time, so the API
# base URL must be supplied as a build arg rather than a runtime env var.
# Defaults to "/api", matching the reverse-proxy route configured in nginx.conf.
ARG VITE_API_BASE_URL=/api
RUN echo "VITE_API_BASE_URL=${VITE_API_BASE_URL}" > .env.production
RUN npm run build

# ---------------------------------------------------------------------------
# Stage 2: Runtime
# nginx:1.27-alpine serves the compiled static assets and doubles as the
# reverse proxy in front of the backend, per the project's architecture.
# ---------------------------------------------------------------------------
FROM nginx:1.27-alpine AS runtime

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -qO- http://localhost:80 || exit 1

CMD ["nginx", "-g", "daemon off;"]
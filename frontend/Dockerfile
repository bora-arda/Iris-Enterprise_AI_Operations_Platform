FROM node:26.13.0-alpine AS build

WORKDIR /app

# CRA inlines REACT_APP_* variables at build time. In Compose the frontend is
# served by nginx and proxies /api to the API service.
ARG REACT_APP_API_BASE_URL=/api
ARG REACT_APP_GRAFANA_URL=http://localhost:3001/d/procurement-overview
ENV REACT_APP_API_BASE_URL=${REACT_APP_API_BASE_URL} \
    REACT_APP_GRAFANA_URL=${REACT_APP_GRAFANA_URL}

COPY package*.json ./
RUN npm ci --no-audit --no-fund

COPY tsconfig.json ./
COPY public/ ./public/
COPY src/ ./src/

RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -q -O /dev/null http://127.0.0.1/ || exit 1

CMD ["nginx", "-g", "daemon off;"]

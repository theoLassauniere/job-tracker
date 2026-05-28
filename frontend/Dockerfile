# syntax=docker/dockerfile:1

# --- Build stage : build Vite via pnpm ---
FROM node:24-alpine AS build
WORKDIR /app
RUN corepack enable
# Cache des dependances : lockfile d'abord
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

# --- Runtime stage : nginx sert les fichiers statiques ---
FROM nginx:1.27-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80

# Phase 1: Use the full Debian-based Bun image (No Alpine!)
FROM oven/bun:1 AS builder

WORKDIR /app

# Accept build arguments
ARG AUTH_ENABLED
ARG AUTH_SECRET
ARG APPWRITE_ENDPOINT
ARG APPWRITE_PROJECT_ID
ARG POCKETBASE_URL
ARG SESSION_MAX_AGE

# Copy EVERYTHING first (crucial so it grabs bun.lock)
COPY . .

# Install dependencies using Bun
RUN bun install

# Build the project
RUN bun run build

# Phase 2: Serve with Nginx
FROM nginx:1.28.2-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose the nginx port
EXPOSE 4173

CMD ["nginx", "-g", "daemon off;"]

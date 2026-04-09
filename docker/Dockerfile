# Phase 1: Build using the full Debian-based Node image (No Alpine compilation errors!)
FROM node:20 AS builder

WORKDIR /app

# Accept build arguments for environment variables
ARG AUTH_ENABLED
ARG AUTH_SECRET
ARG APPWRITE_ENDPOINT
ARG APPWRITE_PROJECT_ID
ARG POCKETBASE_URL
ARG SESSION_MAX_AGE

# Copy package files
COPY package*.json ./

# Install dependencies safely
RUN npm install --legacy-peer-deps

# Copy the rest of the project
COPY . .

# Build the project
RUN npm run build

# Phase 2: Serve with Nginx (Alpine is perfectly fine here since it's just hosting static files)
FROM nginx:1.28.2-alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose the nginx port
EXPOSE 4173

CMD ["nginx", "-g", "daemon off;"]

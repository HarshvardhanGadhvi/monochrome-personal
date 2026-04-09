# Step down to Node 18 (Highly compatible with legacy C++ modules)
FROM node:18 AS builder

WORKDIR /app

ARG AUTH_ENABLED
ARG AUTH_SECRET
ARG APPWRITE_ENDPOINT
ARG APPWRITE_PROJECT_ID
ARG POCKETBASE_URL
ARG SESSION_MAX_AGE

COPY package*.json ./

# Added --loglevel verbose so we can actually see what breaks
RUN npm install --legacy-peer-deps --loglevel verbose

COPY . .

RUN npm run build

FROM nginx:1.28.2-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 4173
CMD ["nginx", "-g", "daemon off;"]

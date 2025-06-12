# Stage 1: Build
FROM node:lts-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html

# Optionales ARG für Umgebungsvariable
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

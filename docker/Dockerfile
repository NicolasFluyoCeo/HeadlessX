# HeadlessX v1.2.0 - MODIFICADO PARA COOLIFY
# Multi-stage build for optimized production image

# Stage 1: Build website
FROM node:20-slim AS website-builder
WORKDIR /app/website
COPY website/package*.json ./
RUN npm ci
COPY website/ ./
RUN npm run build

# Stage 2: Production runtime (SIN NGINX)
FROM mcr.microsoft.com/playwright:v1.50.0-noble

# Instalar solo dependencias necesarias
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies
RUN if [ -f "package-lock.json" ]; then \
        npm ci --only=production; \
    else \
        npm install --production; \
    fi && npm cache clean --force

# Copy modular application source
COPY src/ ./src/

# Copy built website from previous stage
COPY --from=website-builder /app/website/out/ ./website/out/

# Create logs directory
RUN mkdir -p logs

# Create non-root user for security
RUN groupadd -r headlessx && useradd -r -g headlessx headlessx
RUN chown -R headlessx:headlessx /app

# Set user to non-root
USER headlessx

# Environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Expose only the Node.js port
EXPOSE 3000

# Health check apuntando directamente a Node.js en el puerto 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
  CMD curl -f http://localhost:3000/api/health || exit 1

# Start server directamente con Node.js
CMD ["node", "src/server.js"]

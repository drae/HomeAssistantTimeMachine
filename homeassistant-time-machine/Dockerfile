# Optimized Dockerfile for Home Assistant Time Machine
# Multi-stage build for smaller, more secure images

# Stage 1: Dependencies
FROM node:24-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Final image
FROM node:24-alpine AS runtime

# Install security updates and required packages
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
    git \
    tini && \
    rm -rf /var/cache/apk/* /tmp/*

# Create non-root user for security
RUN addgroup -g 2000 -S nodejs && \
    adduser -S nodejs -u 2000

# Create app directory
WORKDIR /app

# Copy dependencies from previous stage
COPY --from=dependencies /app/node_modules ./node_modules

# Copy application files
COPY --chown=nodejs:nodejs . .

# Make run script executable
RUN chmod +x run.sh

# Switch to non-root user
USER nodejs

# Expose the port
EXPOSE 54000

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:54000/health', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) }).on('error', () => process.exit(1))"

# Set environment variables
ENV NODE_ENV=production \
    PORT=54000

# Use tini as init system for proper signal handling
ENTRYPOINT ["/sbin/tini", "--"]

# Start the application
CMD ["./run.sh"]
# ===== BUILD STAGE =====
FROM node:20-bookworm-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY src/ ./src/
COPY tsconfig.json ./
RUN NODE_OPTIONS="--max-old-space-size=4096" npm run build
RUN npm prune --omit=dev

# ===== PRODUCTION STAGE =====
FROM node:20-bookworm-slim AS production
WORKDIR /app
RUN groupadd --gid 1001 appgroup \
 && useradd --uid 1001 --gid appgroup --system --no-create-home appuser
COPY --from=builder /app/node_modules ./node_modules/
COPY --from=builder /app/dist ./dist/
COPY package.json ./
COPY models.yml /models.yml
RUN mkdir -p /data \
 && chown -R appuser:appgroup /data /app \
 && chmod -R go-w /app
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD node -e "require('http').get('http://localhost:8080/health', (res) => process.exit(res.statusCode === 200 ? 0 : 1)).on('error', () => process.exit(1))"
ENTRYPOINT ["npm", "start"]

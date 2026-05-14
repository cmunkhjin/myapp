# ══════════════════════════════════════════
# Stage 1: Dependencies (хамаарал суулгах)
# ══════════════════════════════════════════
FROM node:20-alpine AS deps

WORKDIR /app

# Эхлээд зөвхөн package.json хуулж суулгана
# (code өөрчлөгдөхөд layer cache ашиглана)
COPY package*.json ./
RUN npm ci --omit=dev && \
    npm cache clean --force

# ══════════════════════════════════════════
# Stage 2: Production image (эцсийн image)
# ══════════════════════════════════════════
FROM node:20-alpine AS production

# Аюулгүй байдал: root биш хэрэглэгч үүсгэх
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

# Stage 1-ээс зөвхөн node_modules хуулна
COPY --from=deps /app/node_modules ./node_modules

# Апп код хуулна
COPY server.js ./
COPY package.json ./

# Файлын эзэмшлийг appuser-т шилжүүлэх
RUN chown -R appuser:appgroup /app

# Root биш хэрэглэгчээр ажиллуул
USER appuser

EXPOSE 5000

# Healthcheck нэмэх
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget -qO- http://localhost:5000/health || exit 1

CMD ["node", "server.js"]

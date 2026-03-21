FROM node:20-slim AS builder

WORKDIR /app

COPY package.json .
RUN npm install --omit=dev

FROM node:20-slim

RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY package.json .
COPY worker.js .

RUN chown -R appuser:appgroup /app

USER appuser

CMD ["node", "worker.js"]
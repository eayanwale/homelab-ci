# Needs `output: "standalone"` set in next.config.js first -- that's
# what makes .next/standalone exist, a self-contained server bundle
# with only the production dependencies it actually uses (far smaller
# than copying node_modules wholesale).

FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/public ./public
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static
EXPOSE 3000
CMD ["node", "server.js"]

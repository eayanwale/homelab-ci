# Multi-stage: build with node, serve the static output with nginx.
# Works for any plain static-site build (Vite, CRA, plain HTML/CSS/JS) --
# just point the final COPY at whatever directory your build produces.

FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80

FROM node:16 AS builder
WORKDIR /webapp
copy package*.json ./
COPY . .
RUN npm install
RUN npm run build
FROM nginx
COPY --from=builder /frontend/dist /usr/share/nginx/html

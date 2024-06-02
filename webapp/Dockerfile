FROM node:16 AS builder
RUN mkdir /frontend
WORKDIR /frontend
COPY . .
RUN npm install
RUN npm run build
FROM nginx
COPY --from=builder /frontend/dist /usr/share/nginx/html

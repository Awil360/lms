FROM node:16 AS build
WORKDIR /app
copy package*.json ./
COPY . .
RUN npm install
RUN npm run build
FROM nginx
COPY --from=build /app/dist /usr/share/nginx/html

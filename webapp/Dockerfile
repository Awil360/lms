FROM node:16 AS build
WORKDIR /app
copy package*.json ./
RUN npm install
COPY . .
RUN npm run build
FROM nginx
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80

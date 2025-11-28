
FROM node:16 as build-step
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build


FROM nginx:alpine

COPY --from=build-step /app/dist/angular-15-crud /usr/share/nginx/html

EXPOSE 80
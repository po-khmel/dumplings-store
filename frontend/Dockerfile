# multistage 
### build stage
FROM node:16.20-alpine3.17 as builder

# set wording directory
WORKDIR /app

# copy the application code
COPY ./ ./

# Iistall dependencies
RUN npm install && npx browserslist@latest --update-db

ARG VUE_APP_API_URL=${VUE_APP_API_URL}
ARG NODE_ENV=production

# build 
RUN npm run build


### serve the frontend using NGINX
FROM nginx:latest

# copy the built frontend files
COPY --from=builder /app/dist /usr/share/nginx/html
COPY /nginx/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80

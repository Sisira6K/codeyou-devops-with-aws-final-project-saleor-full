#BUILDER STAGE
#BASE IMAGE
FROM node:18-alpine as builder  
#INSTALL BASH FOR COMPATIBILITY
RUN apk add --no-cache bash
#SET WORKDIR
WORKDIR /app
#MANAGE DEPENDENCIES #COPY PACKAGE FILES AND DEPENDENCIES
COPY package.json package-lock.json ./ 
#AVOID PROMPTS
ENV CI=1
#COMPATABILTY WITH OLDER NODE VERSIONS
RUN npm ci --legacy-peer-deps 
# COPY NECCESSARY DIRECTORIES AND DATA FILES
COPY nginx assets locale scripts src ./
COPY vite.config.js tsconfig.json schema.graphql ./
COPY *.d.ts introspection*.json ./
COPY .featureFlags/ ./featureFlags/
#CONFIGURE ENVIRONMENT VARIABLES
ARG API_URL
ARG APP_MOUNT_URI
ARG APPS_MARKETPLACE_API_URL
ARG APPS_TUNNEL_URL_KEYWORDS
ARG STATIC_URL
ARG SKIP_SOURCEMAPS
ARG LOCALE_CODE
#BIND ARG VALUES TO ENVIRONMENT VARIABLES WITH DEFAULT VALUES
ENV API_URL=${API_URL:-http://localhost:8000/graphql/} \
    APP_MOUNT_URI=${APP_MOUNT_URI:-/dashboard/} \
    APPS_MARKETPLACE_API_URL=${APPS_MARKETPLACE_API_URL:-https://apps.saleor.io/api/v2/saleor-apps} \
    APPS_TUNNEL_URL_KEYWORDS=${APPS_TUNNEL_URL_KEYWORDS} \
    STATIC_URL=${STATIC_URL:-/dashboard/} \
    SKIP_SOURCEMAPS=${SKIP_SOURCEMAPS:-true} \
    LOCALE_CODE=${LOCALE_CODE:-EN}

#BUILD THE APPLICATION
RUN npm run build

#THE RUNNER STAGE
#BASE IMAGE
FROM nginx:stable-alpine as runner 
#SET WORKDIR
WORKDIR /app

#COPY THE BUILT APPLICATION FROM THE BUILDER STAGE
COPY --from=builder /app/build/ /app/
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY ./nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh
#SET THE PORT
EXPOSE 80
#SET THE ENTRYPOINT
ENTRYPOINT ["nginx", "-g", "daemon off;"]



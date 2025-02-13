#BUILDER STAGE
#BASE IMAGE
FROM node:18-alpine AS builder  
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
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh
COPY assets/ /app/assets/
COPY locale/ /app/locale/
COPY scripts/ /app/scripts/
COPY src/ /app/src/
COPY vite.config.js /app/vite.config.js
COPY tsconfig.json /app/tsconfig.json
COPY schema.graphql /app/schema.graphql
COPY *.d.ts /app/types/
COPY introspection.json /app/introspection.json
COPY .featureFlags/ /app/.featureFlags/

#CONFIGURE ENVIRONMENT VARIABLES
ARG API_URL=http://localhost:8000/graphql/
ARG APP_MOUNT_URI=/dashboard/
ARG APPS_MARKETPLACE_API_URL=https://apps.saleor.io/api/v2/saleor-apps
ARG APPS_TUNNEL_URL_KEYWORDS=
ARG STATIC_URL=/dashboard/
ARG SKIP_SOURCEMAPS=true
ARG LOCALE_CODE=EN
 #SET ENVIRONMENT VARIABLES
ENV API_URL=${API_URL}
ENV APP_MOUNT_URI=${APP_MOUNT_URI}
ENV APPS_MARKETPLACE_API_URL=${APPS_MARKETPLACE_API_URL}
ENV APPS_TUNNEL_URL_KEYWORDS=${APPS_TUNNEL_URL_KEYWORDS}
ENV STATIC_URL=${STATIC_URL}
ENV SKIP_SOURCEMAPS=${SKIP_SOURCEMAPS}
ENV LOCALE_CODE=${LOCALE_CODE}

#BUILD THE APPLICATION
RUN npm run build 

#THE RUNNER STAGE
#BASE IMAGE
FROM nginx:stable-alpine AS runner 
#SET WORKDIR
WORKDIR /app/

#COPY THE BUILT APPLICATION FROM THE BUILDER STAGE
COPY --from=builder /app/build/ /app/
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY ./nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh

RUN chmod +x /docker-entrypoint.d/50-replace-env-vars.sh
#SET THE PORT
EXPOSE 80
#SET THE ENTRYPOINT
ENTRYPOINT ["nginx", "-g", "daemon off;"]
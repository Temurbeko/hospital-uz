# syntax=docker/dockerfile:1.3
#--------------------------------------------
# Dev Stage - Assembles and Builds Frontend
#--------------------------------------------
FROM --platform=$BUILDPLATFORM node:18-alpine as dev

ARG APP_SHELL_VERSION=next

RUN mkdir -p /app 
WORKDIR /app


COPY packages/openmrs-esm-login-app-6.0.4.tgz .
COPY packages/openmrs-esm-patient-attachments-app-9.1.0.tgz .
COPY packages/openmrs-esm-patient-banner-app-9.1.0.tgz .
COPY packages/openmrs-esm-patient-medications-app-9.1.0.tgz .
COPY packages/openmrs-esm-patient-orders-app-9.1.0.tgz .
COPY packages/openmrs-esm-patient-tests-app-9.1.0.tgz .
COPY packages/openmrs-esm-patient-vitals-app-9.1.0.tgz .

COPY spa-assemble-config.json .
COPY spa-build-config.json .

RUN yarn add ./openmrs-esm-login-app-6.0.4.tgz
RUN yarn add ./openmrs-esm-patient-attachments-app-9.1.0.tgz
RUN yarn add ./openmrs-esm-patient-banner-app-9.1.0.tgz
RUN yarn add ./openmrs-esm-patient-medications-app-9.1.0.tgz
RUN yarn add ./openmrs-esm-patient-orders-app-9.1.0.tgz
RUN yarn add ./openmrs-esm-patient-tests-app-9.1.0.tgz
RUN yarn add ./openmrs-esm-patient-vitals-app-9.1.0.tgz

  


ARG CACHE_BUST
RUN npx --legacy-peer-deps openmrs@${APP_SHELL_VERSION:-next} assemble --manifest --mode config --config spa-assemble-config.json --target ./spa
RUN npx --legacy-peer-deps openmrs@${APP_SHELL_VERSION:-next} build --build-config spa-build-config.json --target ./spa
RUN if [ ! -f ./spa/index.html ]; then echo 'Build failed. Please check the logs above for details. This may have happened because of an update to a library that OpenMRS depends on.'; exit 1; fi

#--------------------------------------------
# Runtime Stage - Published Image
#--------------------------------------------
FROM nginx:1.25-alpine

RUN apk update && \
    apk upgrade && \
    # add more utils for sponge and envsubst
    apk add --no-cache moreutils

# clear any default files installed by nginx
RUN rm -rf /usr/share/nginx/html/*

COPY startup.sh /usr/local/bin/startup.sh
RUN chmod +x /usr/local/bin/startup.sh

COPY nginx.conf /etc/nginx/nginx.conf

COPY --from=dev /app/spa /usr/share/nginx/html
COPY config-core_demo.json /usr/share/nginx/html
COPY src/main/resources/logo.png /usr/share/nginx/html
COPY src/main/resources/login-logo.png /usr/share/nginx/html
COPY src/main/resources/patient-chart-logo.png /usr/share/nginx/html
COPY src/main/resources/favicon.ico /usr/share/nginx/html


CMD ["/usr/local/bin/startup.sh"]

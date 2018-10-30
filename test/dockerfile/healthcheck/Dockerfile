FROM nginx:1.14.0-alpine
LABEL Author="FENNG <YINF@IMOVIE.CN>"

RUN apk add --no-cache curl
COPY index.html /usr/share/nginx/html/
HEALTHCHECK --interval=15s --timeout=5s --start-period=10s CMD curl -f http://localhost/ || exit 1

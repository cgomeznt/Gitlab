# FROM node:12-alpine
FROM 10.134.0.252:4443/nodejs12alpine
# Crea el directorio para el volumen persistente
RUN mkdir -p /opt/fotos
# Monta el volumen persistente en /opt/fotos
VOLUME /opt/fotos
WORKDIR /app
COPY app/. .
RUN yarn install --production
# Configura la aplicación para usar /opt/fotos
ENV PHOTOS_DIR=/opt/fotos
CMD ["node", "src/index.js"]




### STAGE 1: Build ###
FROM node AS build
WORKDIR  /usr/src/app
COPY package*.json ./

RUN npm install -g @angular/cli
RUN npm install
COPY . .
RUN ng build

### STAGE 2: Run ###
FROM httpd
COPY --from=build /usr/src/app/myfile /usr/local/apache2/htdocs/
COPY --from=build /usr/src/app/dist/* /usr/local/apache2/htdocs/

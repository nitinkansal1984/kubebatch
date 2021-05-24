## pulling the ubuntu based image from dockerhub
FROM ubuntu
MAINTAINER demouser@gmail.com
USER root
 ## update the packages to the latest
RUN apt-get update -y 
## install nginx
RUN apt-get install nginx -y 
COPY ./index.html /var/www/html/
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
CMD ["nginx"]


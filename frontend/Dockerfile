# Use nginx base image
FROM httpd:2.4

# Copy index.html to nginx default html directory
COPY ./index.html /usr/local/apache2/htdocs/

COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
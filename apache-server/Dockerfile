FROM php:apache
RUN useradd -s /bin/bash apache 
USER apache
COPY index.html /var/www/html/
COPY info.php /var/www/html/
EXPOSE 80

# Use official PHP image with Apache
FROM php:8.0-apache

# Set working directory
WORKDIR /var/www/html

# Copy application source code
COPY ./src/ .

# Enable Apache rewrite module
RUN a2enmod rewrite

# Expose port 80
EXPOSE 80

# Start Apache when container runs
CMD ["apache2-foreground"]

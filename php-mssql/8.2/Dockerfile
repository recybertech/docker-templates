FROM --platform=linux/amd64 php:8.2-fpm

# Set working directory
WORKDIR /var/www/html/

USER root

# Install dependencies for the operating system software
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    gnupg2 \
    curl \
    supervisor

RUN curl -sSL https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions -o - | sh -s \
      ldap gd exif zip

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Microsoft SQL Server Prerequisites & SQL Server drivers
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/11/prod.list \
        > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \ 
    && ACCEPT_EULA=Y apt-get install -y msodbcsql17 unixodbc-dev

RUN pecl install sqlsrv pdo_sqlsrv

# Install composer (php package manager)
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Expose port 9000 and start php-fpm server (for FastCGI Process Manager)
EXPOSE 9000
CMD ["php-fpm"]
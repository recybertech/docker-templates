FROM php:8.5-fpm

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
    supervisor \
    calendar

# PHP extensions
RUN curl -sSL https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions \
        -o /usr/local/bin/install-php-extensions \
    && chmod +x /usr/local/bin/install-php-extensions \
    && install-php-extensions ldap gd exif zip

# Microsoft ODBC Driver 18, which sqlsrv/pdo_sqlsrv link against. The suite is
# derived from the base image so this keeps working across Debian upgrades, and
# both signing keys are loaded (bullseye/bookworm use the old one, trixie the
# 2025 one). Driver 17 is amd64-only, so 18 is required for native arm64 builds.
RUN . /etc/os-release \
    && { curl -fsSL https://packages.microsoft.com/keys/microsoft.asc; \
         curl -fsSL https://packages.microsoft.com/keys/microsoft-2025.asc; } \
        | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/${VERSION_ID}/prod ${VERSION_CODENAME} main" \
        > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql18 unixodbc-dev

# sqlsrv/pdo_sqlsrv from pinned tarballs. pecl.php.net only advertises REST 1.0
# and 1.1, so `pecl install sqlsrv` cannot resolve a release and fails with
# "No releases available"; handing pecl the tarball skips that lookup entirely.
ARG SQLSRV_VERSION=5.13.3
RUN curl -fsSL https://pecl.php.net/get/sqlsrv-${SQLSRV_VERSION}.tgz -o /tmp/sqlsrv.tgz \
    && curl -fsSL https://pecl.php.net/get/pdo_sqlsrv-${SQLSRV_VERSION}.tgz -o /tmp/pdo_sqlsrv.tgz \
    && pecl install /tmp/sqlsrv.tgz /tmp/pdo_sqlsrv.tgz \
    && docker-php-ext-enable sqlsrv pdo_sqlsrv \
    && rm -f /tmp/sqlsrv.tgz /tmp/pdo_sqlsrv.tgz

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install composer (php package manager). Copied from the official multi-arch
# composer image instead of the installer script, which fails on PHP 8.5 with
# "Failed to decode zlib stream" when it downloads composer.phar.
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

# Expose port 9000 and start php-fpm server (for FastCGI Process Manager)
EXPOSE 9000
CMD ["php-fpm"]
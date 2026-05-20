Berikut setup production-ready untuk CodeIgniter 4 menggunakan:

* PHP 8.2 FPM
* Nginx
* MySQL/MariaDB
* Redis (optional)
* Coolify compatible
* Optimized untuk production

Struktur project:

```txt
project-root/
├── app/
├── public/
├── writable/
├── docker/
│   ├── nginx/
│   │   └── default.conf
│   └── php/
│       └── local.ini
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
└── .env
```

---

# Dockerfile

```dockerfile
FROM php:8.2-fpm-alpine

# Install system dependencies
RUN apk add --no-cache \
    bash \
    git \
    curl \
    unzip \
    icu-dev \
    oniguruma-dev \
    libzip-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev

# Install PHP extensions
RUN docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg && \
    docker-php-ext-install \
    pdo \
    pdo_mysql \
    mysqli \
    intl \
    mbstring \
    zip \
    exif \
    gd

# Install Redis extension
RUN pecl install redis && docker-php-ext-enable redis

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy app
COPY . .

# Install dependencies
RUN composer install \
    --no-dev \
    --optimize-autoloader

# Set permissions
RUN chown -R www-data:www-data writable

# Copy PHP config
COPY docker/php/local.ini /usr/local/etc/php/conf.d/local.ini

# Expose port
EXPOSE 9000

CMD ["php-fpm"]
```

---

# docker-compose.yml

```yaml
version: '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ci4_app
    restart: unless-stopped
    working_dir: /var/www/html
    volumes:
      - ./:/var/www/html
    networks:
      - ci4

  nginx:
    image: nginx:stable-alpine
    container_name: ci4_nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./:/var/www/html
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - ci4

  db:
    image: mariadb:11
    container_name: ci4_db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ci4
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_USER: ci4user
      MYSQL_PASSWORD: ci4password
    volumes:
      - dbdata:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - ci4

  redis:
    image: redis:7-alpine
    container_name: ci4_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      - ci4

networks:
  ci4:

volumes:
  dbdata:
```

---

# Nginx Config

File:
`docker/nginx/default.conf`

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/html/public;
    index index.php index.html;

    client_max_body_size 100M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

---

# PHP Production Config

File:
`docker/php/local.ini`

```ini
memory_limit=512M
upload_max_filesize=100M
post_max_size=100M
max_execution_time=300
date.timezone=Asia/Jakarta
display_errors=Off
log_errors=On
```

---

# .dockerignore

```txt
.git
vendor
node_modules
docker-compose.yml
.env
```

---

# CI4 .env Production

```env
CI_ENVIRONMENT = production

app.baseURL = 'https://yourdomain.com/'

database.default.hostname = db
database.default.database = ci4
database.default.username = ci4user
database.default.password = ci4password
database.default.DBDriver = MySQLi

cache.handler = redis
cache.backupHandler = file
redis.host = redis
redis.port = 6379
```

---

# Jalankan Local

```bash
docker compose up -d --build
```

Akses:

* App → `http://localhost`
* Database → `localhost:3306`

---

# Deploy ke Coolify

Di [Coolify](https://coolify.io?utm_source=chatgpt.com):

1. Push project ke GitHub/GitLab
2. New Resource → Docker Compose
3. Connect repository
4. Paste atau auto-detect `docker-compose.yml`
5. Tambahkan domain
6. Enable SSL otomatis
7. Deploy

---

# Optimasi Production Tambahan

Disarankan:

## 1. Cache Route

```bash
php spark optimize
```

## 2. Gunakan OPcache

Tambahkan di `local.ini`:

```ini
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000
```

## 3. Jangan mount volume di production

Di production sebaiknya:

```yaml
volumes:
  - ./:/var/www/html
```

DIHAPUS agar immutable container.

---

# Versi Production Lebih Aman

Untuk production final:

* gunakan multi-stage Docker build,
* non-root user,
* healthcheck,
* queue worker,
* scheduler/cron,
* separate network,
* secrets management.

Kalau mau, saya juga bisa bantu buatkan:

* versi super optimal multi-stage build,
* CI4 + RoadRunner,
* CI4 + FrankenPHP,
* auto deploy GitHub Actions → Coolify,
* queue worker & cron,
* setup HTTPS Cloudflare,
* atau stack high-performance untuk VPS kecil 1GB RAM.

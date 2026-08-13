# LARAVEL APACHE


## Apache Virtualisasi
```
<VirtualHost *:8081>
    ServerAdmin pudinilham@gmail.com

    DocumentRoot /var/www/public/career/public

    <Directory /var/www/public/career/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # PHP 8.4-FPM
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    ErrorLog ${APACHE_LOG_DIR}/career-8081-error.log
    CustomLog ${APACHE_LOG_DIR}/career-8081-access.log combined
</VirtualHost>
```


### Pudin Saepudin

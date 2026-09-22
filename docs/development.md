# Minimal development setup

This setup reproduces the current legacy application before modernization.

Local development only. Do not expose this stack publicly.

## 1. Required components

Runtime:
- nginx
- PHP 7.3
- Composer 2
- MariaDB
- Elasticsearch 5
- Memcached

Asset build:
- Node.js 10
- Less 3.13
- Java
- ImageMagick

Optional services are not required initially:
- RabbitMQ
- WebSockets
- SMTP delivery
- Chromium

## 2. Clone

```bash
sudo mkdir -p /var/www/dev.lairdubois.fr
sudo chown "$USER":www-data /var/www/dev.lairdubois.fr

git clone https://github.com/MichelDwo/lairdubois.git /var/www/dev.lairdubois.fr
cd /var/www/dev.lairdubois.fr
git checkout p0.1-reproducible-dev-environment
```

## 3. Configure

```bash
cp app/config/parameters.yml.dist app/config/parameters.yml
```

Set at least:
- MariaDB credentials
- Elasticsearch host/port
- Memcached host/port
- a local `secret`

RabbitMQ, WebSocket and external API values can remain dummy local values.

Generate a development DKIM key because mail code expects the file even when delivery is not used:

```bash
mkdir -p keys
openssl genrsa -out keys/private.pem 1024
```

## 4. Install PHP dependencies

```bash
composer install
```

Do not run `composer update` during P0.1/P0.2.

## 5. Initialize MariaDB

```bash
php bin/console doctrine:database:create --env=dev
php bin/console doctrine:schema:update --force --env=dev
mysql db_fr_lairdubois_www < docs/database/schema-sessions.sql
```

## 6. Build application assets

```bash
php bin/console assetic:dump --env=dev
php bin/console assets:install web --env=dev
mkdir -p uploads
cp src/Ladb/CoreBundle/Resources/fixtures/empty*.png uploads/
```

## 7. Initialize Elasticsearch

```bash
php bin/console fos:elastica:populate --env=dev
```

## 8. Configure nginx

Use:

```
docs/nginx/conf/dev.lairdubois.fr.conf
```

The current configuration expects:
- project: `/var/www/dev.lairdubois.fr`
- PHP-FPM socket: `/run/php/php7.3-fpm.sock`

Add to `/etc/hosts`:

```
127.0.0.1 dev.lairdubois.fr
```

Restart nginx and PHP-FPM.

## 9. Validate

Open:

```
http://dev.lairdubois.fr
```

Minimum validation:
- homepage loads
- CSS/JS load
- database-backed pages load
- search/list pages load
- a basic create/update action works

Expected degraded features:
- no asynchronous view counters
- no Web Push
- no realtime workflow updates
- no outgoing email delivery
- no automatic shared-link screenshots

## Next step

Run this procedure on a clean environment and record every failure or undocumented dependency. That validation is P0.2.

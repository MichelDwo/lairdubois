# Minimal development setup

This setup reproduces the current legacy application before modernization.

Local development only. Do not expose this stack publicly.

## 1. Prepare Debian 10

Debian 10 is archived. If `apt update` no longer works with the configured mirrors, use:

```bash
sudo tee /etc/apt/sources.list >/dev/null <<'EOF'
deb http://archive.debian.org/debian buster main contrib non-free
EOF

sudo apt-get -o Acquire::Check-Valid-Until=false update
```

## 2. Install system dependencies

```bash
sudo apt-get -o Acquire::Check-Valid-Until=false install -y \
  ca-certificates curl wget gnupg apt-transport-https git unzip openssl \
  nginx mariadb-server mariadb-client memcached \
  php7.3 php7.3-cli php7.3-fpm php7.3-curl php7.3-intl \
  php7.3-gd php7.3-imagick php7.3-mysql php7.3-mbstring \
  php7.3-xml php7.3-zip php7.3-bz2 php7.3-gmp php7.3-bcmath \
  php-memcached \
  nodejs npm \
  imagemagick ghostscript librsvg2-bin pngquant optipng jpegoptim
```

Install Less:

```bash
sudo npm install -g less@3.13.1
```

Install Composer 2:

```bash
curl -sS https://getcomposer.org/installer | sudo php -- \
  --2 --install-dir=/usr/local/bin --filename=composer
```

Check:

```bash
php -v
node --version
lessc --version
composer --version
```

Expected baseline:
- PHP 7.3
- Node.js 10
- Less 3.13
- Composer 2

## 3. Install Java 8

Elasticsearch 5.6 requires Java 8. Debian 10's default JRE is Java 11, so install Java 8 separately.

```bash
curl -L \
  'https://api.adoptium.net/v3/binary/latest/8/ga/linux/x64/jre/hotspot/normal/eclipse' \
  -o /tmp/temurin8.tar.gz

sudo mkdir -p /opt/java8
sudo tar -xzf /tmp/temurin8.tar.gz -C /opt/java8 --strip-components=1

/opt/java8/bin/java -version
```

## 4. Install Elasticsearch 5.6

```bash
wget -O /tmp/elasticsearch-5.6.16.deb \
  https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.16.deb

sudo dpkg -i /tmp/elasticsearch-5.6.16.deb

echo 'JAVA_HOME=/opt/java8' | sudo tee -a /etc/default/elasticsearch

sudo systemctl daemon-reload
sudo systemctl enable --now elasticsearch
```

For a small VM, limit the Elasticsearch heap:

```bash
sudo sed -i 's/^-Xms2g/-Xms1g/; s/^-Xmx2g/-Xmx1g/' /etc/elasticsearch/jvm.options
sudo systemctl restart elasticsearch
```

Check:

```bash
curl http://localhost:9200/
```

## 5. Clone

```bash
sudo mkdir -p /var/www/dev.lairdubois.fr
sudo chown "$USER":www-data /var/www/dev.lairdubois.fr

git clone https://github.com/MichelDwo/lairdubois.git /var/www/dev.lairdubois.fr
cd /var/www/dev.lairdubois.fr
git checkout p0.1-reproducible-dev-environment
```

## 6. Configure

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

## 7. Install PHP dependencies

```bash
composer install
```

Do not run `composer update` during P0.1/P0.2.

## 8. Initialize MariaDB

```bash
php bin/console doctrine:database:create --env=dev
php bin/console doctrine:schema:update --force --env=dev
mysql db_fr_lairdubois_www < docs/database/schema-sessions.sql
```

## 9. Build application assets

```bash
php bin/console assetic:dump --env=dev
php bin/console assets:install web --env=dev
mkdir -p uploads
cp src/Ladb/CoreBundle/Resources/fixtures/empty*.png uploads/
```

## 10. Initialize Elasticsearch

```bash
php bin/console fos:elastica:populate --env=dev
```

## 11. Configure nginx

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

## 12. Validate

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

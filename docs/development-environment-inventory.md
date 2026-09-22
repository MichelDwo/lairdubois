# Development environment inventory

## Core runtime
- Linux
- nginx
- PHP >= 7.3
- Composer
- MariaDB / MySQL (pdo_mysql)

## Required PHP extensions
- curl
- dom
- imagick
- exif
- json
- memcached
- libxml
- fileinfo
- iconv
- intl
- gd
- mbstring
- xml
- zip
- bz2
- gmp
- bcmath

## Application services
- MariaDB — database
- Elasticsearch 5 — search
- RabbitMQ — background jobs
- Memcached — cache
- WebSocket server — realtime features
- SMTP server — email

## Asset build
- Node.js <= 10
- Less 3.13
- Java
- Closure Compiler
- YUI Compressor

## Media / auxiliary tools
- ImageMagick
- Ghostscript
- librsvg
- pngquant
- optipng
- jpegoptim
- Chromium — shared-link screenshots

## External integrations
Not required for a basic local startup:
- Google API
- Facebook
- Twitter
- Pinterest
- Mastodon
- Stripe
- Web Push
- Google Analytics

## Next check
Determine which of the following can be disabled for a basic development instance:
- Elasticsearch
- RabbitMQ
- Memcached
- WebSockets
- SMTP
- Chromium

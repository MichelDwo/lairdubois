# Development environment inventory

## Minimal usable development environment

### Required
- Linux
- nginx
- PHP >= 7.3
- Composer
- MariaDB / MySQL
- Elasticsearch 5

MariaDB is required by Doctrine and also stores Symfony sessions.

Elasticsearch is used by search and many entity listings. The application can boot without it, but important pages will fail or become unusable.

### Required for write operations
- Memcached

Memcached is used by Symfony Lock for non-blocking locks in create/update actions. Read-only browsing can work without it, but normal development should currently include it.

### Optional / degraded mode
- RabbitMQ
  - Used for view counters and Web Push queues.
  - Producer failures are caught, so basic application requests continue to work.
- WebSocket server
  - Used for realtime workflow features.
- SMTP
  - Not required in development.
  - Swiftmailer uses a file spool; actual delivery can remain disabled.
- Chromium
  - Only required for shared-link screenshot generation.

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

## Asset build
- Node.js <= 10
- Less 3.13
- Java
- Closure Compiler
- YUI Compressor

These are required to rebuild the current Assetic CSS/JS assets.

## Media / auxiliary tools
- ImageMagick
- Ghostscript
- librsvg
- pngquant
- optipng
- jpegoptim

## External integrations
Not required for basic local development:
- Google API
- Facebook
- Twitter
- Pinterest
- Mastodon
- Stripe
- Web Push
- Google Analytics

## Current minimal target

For a reasonably functional development instance:

```
PHP + nginx + MariaDB + Elasticsearch + Memcached
```

RabbitMQ, WebSockets, SMTP and Chromium can initially remain disabled.

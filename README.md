# kwik-nginx

Drop-in nginx configuration for [kwik.gg](https://kwik.gg) (with legacy redirects from kwik.zip) - zero-knowledge encrypted file sharing.

## Quick Deploy

```bash
git clone https://github.com/Peckage/kwik-nginx.git
cd kwik-nginx
sudo ./deploy.sh
```

That's it. The script:

1. Copies `conf.d/*.conf` files (rate limiting, performance, upstream)
2. Copies site config to `sites-available/kwik.zip` (serves kwik.gg and redirects kwik.zip)
3. Enables the site
4. Tests and reloads nginx

**No nginx.conf modifications required** - works as an addon to any existing nginx setup.

## What Gets Installed

```text
/etc/nginx/
├── conf.d/
│   ├── kwik-ratelimit.conf    # Rate limiting zones
│   ├── kwik-performance.conf  # Timeouts & buffers for large files
│   ├── kwik-upstream.conf     # Upstream to Next.js app
│   └── kwik-compression.conf  # Gzip/Brotli compression
├── sites-available/
│   └── kwik.gg                # Main site config (kwik.gg primary, kwik.zip redirects)
└── sites-enabled/
    └── kwik.gg -> ../sites-available/kwik.gg
```

## Features

- **Unlimited file uploads** - `client_max_body_size 0`
- **2-hour timeouts** - for 50GB+ file transfers
- **Resumable downloads** - `proxy_force_ranges on`
- **Streaming uploads** - no buffering, chunked transfer
- **Zero-knowledge security headers** - strict CSP, HSTS, no referrer leakage
- **Rate limiting** - DDoS protection with generous limits for file transfers

## SSL Setup

After deploying, if you don't have SSL certs:

```bash
sudo certbot certonly --webroot -w /var/www/kwik.gg -d kwik.gg -d www.kwik.gg

For legacy redirects (optional, only if you own kwik.zip):
sudo certbot certonly --webroot -w /var/www/kwik.gg -d kwik.zip -d www.kwik.zip
```

## Manual Install

If you prefer not to use the script:

```bash
# Copy conf.d files
sudo cp conf.d/kwik-*.conf /etc/nginx/conf.d/

# Copy and enable site
sudo cp sites-available/kwik.gg /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/kwik.gg /etc/nginx/sites-enabled/

# Test and reload
sudo nginx -t && sudo systemctl reload nginx
```

## Uninstall

```bash
sudo rm /etc/nginx/conf.d/kwik-*.conf
sudo rm /etc/nginx/sites-available/kwik.gg
sudo rm /etc/nginx/sites-enabled/kwik.gg
sudo nginx -t && sudo systemctl reload nginx
```

## Requirements

- Nginx with `sites-available`/`sites-enabled` structure (Debian/Ubuntu style)
- Or any nginx that includes `conf.d/*.conf`
- Next.js app running on `127.0.0.1:3000`

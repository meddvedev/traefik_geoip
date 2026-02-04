# MaxMind GeoIP traefik plugin & supports localized GeoIP data

> This project is a fork of  
> [Maronato/traefik_geoip](https://github.com/Maronato/traefik_geoip),  
> originally based on  
> [GiGInnovationLabs/traefikgeoip2](https://github.com/GiGInnovationLabs/traefikgeoip2).
>
> Licensed under the Apache License 2.0.

This is a modified version of `traefik_geoip` with the following changes:

- Support for **localized GeoIP data** (e.g. `de`, `en` etc...) using MaxMind `names[lang]`
- Logs are hidden by default and can be enabled with `debug: true`
- Adds latitude, longitude, geohash, and country name  
  (country code is exposed separately as `CountryCode`)
- Removes the `X-` prefix from header names, per  
  [RFC 6648](https://www.rfc-editor.org/rfc/rfc6648)
- Adds support for `excludeIPs` (IPs and CIDRs excluded from GeoIP checks)
- Does not add headers if their values cannot be determined
- Resolves client IP by checking:
  - `X-Forwarded-For` first
  - `req.RemoteAddr` as fallback
- Optional `setRealIP: true` flag to reset `X-Real-IP` to the resolved client IP

Supported languages:
- de: German
- en: English
- es: Spanish
- fr: French
- ja: Japanese
- pt-BR: Brazilian Portuguese
- ru: Russian
- zh-CN: Simplified Chinese

---

[Traefik](https://doc.traefik.io/traefik/) plugin  
that registers a custom middleware  
for getting data from  
[MaxMind GeoIP databases](https://www.maxmind.com/en/geoip2-services-and-databases)  
and passing it downstream via HTTP request headers.

Supports both:

- [GeoIP2](https://www.maxmind.com/en/geoip2-databases)
- [GeoLite2](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data)

## Usage
cli:
```
services:
  traefik:
    image: traefik
    volumes:
      - ./GeoLite2-City.mmdb:/var/lib/geolite/GeoLite2-City.mmdb:ro
    command:
      - "--experimental.plugins.geoip.modulename=github.com/meddvedev/traefik_geoip"
      - "--experimental.plugins.geoip.version=v0.0.1"
    labels:
      - 'traefik.enable=true'
      # geoip middleware
      - 'traefik.http.middlewares.geoip.plugin.geoip.dbpath=/var/lib/geolite/GeoLite2-City.mmdb'
      - 'traefik.http.middlewares.geoip.plugin.geoip.lang=de'
```

yml:
```
http:
  middlewares:
    geoip:
      plugin:
        geoip:
          dbpath: /var/lib/geolite/GeoLite2-City.mmdb
          lang: de
```

```
Output
{
  "GeoIP-Country": "Deutschland"
  "GeoIP-City": "München"
}

If lang does not provide output will be in english
```

### Local dev via docker

Extract geolite2 `tar -xf geolite2.tgz`

Start local dev container
```
docker run --rm -it \
  -v $(pwd):/app \
  -w /app \
  --name traefik-geoip-dev \
  golang:1.22-alpine \
  sh
```
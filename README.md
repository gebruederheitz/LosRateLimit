# Rate Limit Middleware for PHP

[![Build Status](https://travis-ci.org/Lansoweb/LosRateLimit.svg?branch=master)](https://travis-ci.org/Lansoweb/LosRateLimit) [![Latest Stable Version](https://poser.pugx.org/los/los-rate-limit/v/stable.svg)](https://packagist.org/packages/los/los-rate-limit) [![Total Downloads](https://poser.pugx.org/los/los-rate-limit/downloads.svg)](https://packagist.org/packages/los/los-rate-limit)

LosRateLimit is a php middleware to implement a rate limit.

First, the middleware will look for an X-Api-Key header to use as key. If not found, it will fall back to the remote IP.

Each one has its own limits (see configuration below).

Attention! This middleware does not validate the Api Key, you must add a middleware before this one to validate it.

## Requirements

* PHP >= 7.4
* Psr\SimpleCache implementation

## Installation

```bash
composer require los/los-rate-limit
```

### Configuration
```php
'los_rate_limit' => [
    'max_requests' => 100,
    'reset_time' => 3600,
    'ip_max_requests' => 100,
    'ip_reset_time' => 3600,
    'api_header' => 'X-Api-Key',
    'trust_forwarded' => false,
    'prefer_forwarded' => false,
    'forwarded_headers_allowed' => [
        'Client-Ip',
        'Forwarded',
        'Forwarded-For',
        'X-Cluster-Client-Ip',
        'X-Forwarded',
        'X-Forwarded-For',
    ],
    'forwarded_ip_index' => null,
    'headers' => [
        'limit' => 'X-RateLimit-Limit',
        'remaining' => 'X-RateLimit-Remaining',
        'reset' => 'X-RateLimit-Reset',
    ],
    'keys' => [
        'b9155515728fa0f69d9770f7877cb50a' => [
            'max_requests' => 100,
            'reset_time' => 3600,
        ],
    ],
    'ips' => [
        '127.0.0.1' => [
            'max_requests' => 100,
            'reset_time' => 3600,
        ],
    ],
    'hash_ips' => false,
    'hash_salt' => 'Los%Rate',
]
```

* `max_requests` How many requests are allowed before the reset time (using API Key)
* `reset_time` After how many seconds the counter will be reset (using API Key)
* `ip_max_requests` How many requests are allowed before the reset time (using remote IP Key)
* `ip_reset_time` After how many seconds the counter will be reset (using remote IP Key)
* `api_header` Header name to get the api key from.
* `trust_forwarded` If the X-Forwarded (and similar) headers and be trusted. If not, only $_SERVER['REMOTE_ADDR'] will be used.
* `prefer_forwarded` Whether forwarded headers should be used in preference to the remote address, e.g. if all requests are forwarded through a routing component or reverse proxy which adds these headers predictably. This is a bad idea unless your app can **only** be reached this way.
* `forwarded_headers_allowed` An array of strings which are headers you trust to contain source IP addresses.
* `forwarded_ip_index` If null (default), the first plausible IP in an XFF header (reading left to right) is used. If numeric, only a specific index of IP is used. Use `-2` to get the penultimate IP from the list, which could make sense if the header always ends `...<client_ip>, <router_ip>`. Or use `0` to use only the first IP (stopping if it's not valid). Like `prefer_forwarded`, this only makes sense if your app's always reached through a predictable hop that controls the header - remember these are easily spoofed on the initial request.
* `keys` Specify different max_requests/reset_time per api key
* `ips` Specify different max_requests/reset_time per IP
* `hash_ips` Enable the hashing of IP addresses before storing them. This is particularly useful when using a 
  filesystem-based cache implementation and working with IPv6 addresses. A salted MD5-hash will be used if you set 
  this to `true`.
* `hash_salt' This setting allows you to optionally define a custom salt when using hashed IP addresses. Only 
  effective when `hash_ips` is `true`.

The values above indicate that the user can trigger 100 requests per hour.

If you want to disable ip access (e.g. allowing just access via X-Api-Key), just set ip_max_requests to 0 (zero).

## Usage

Just add the middleware as one of the first middlewares.

The provided factory uses the container to get a \Psr\SimpleCache\CacheInterface (PSR-16). 
Most implementations provide both PSR-6 and PSR-16, or at least a decorator.
Recommended: [zend-cache](https://github.com/laminas/laminas-cache) or [symfony/cache](https://github.com/symfony/cache).

### Zend Expressive

If you are using [expressive-skeleton](https://github.com/mezzio/mezzio-skeleton),
you can copy `config/los-rate-limit.local.php.dist` to
`config/autoload/los-rate-limit.local.php` and modify configuration as your needs.

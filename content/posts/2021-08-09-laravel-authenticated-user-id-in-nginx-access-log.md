---
title: Laravel authenticated user ID in NGINX access log
slug: laravel-authenticated-user-id-in-nginx-access-log
author: Alex Bouma
tags:
  - laravel
  - php
  - guide
date: 2021-08-09T16:00:00+02:00
lastmod: 2021-08-09T16:00:00+02:00
description: "Let's add the authenticated user ID to the NGINX access log for easier debugging."
summary: "Let's add the authenticated user ID to the NGINX access log for easier debugging."
keywords:
  - laravel
  - nginx
  - logging
draft: false
cover: /img/ghost/unsplash-logs.jpg
---

For debugging and/or easy searching it's handy to have the user ID in the [NGINX](https://nginx.org/) access logs so you can quickly find logs for a specific user.

There are 2 parts to achieving this:

1. Add the user ID as a response header to our Laravel app using a middleware
2. Add the user ID as part of the NGINX log format and use that log format for the access logs

## Adding the user ID as a response header

It makes the most sense to use a middleware for this purpose and we can make it really simple thanks to Laravel and PHP 8 syntax.

Create the `app/Http/Middleware/AppendUserIdToResponse.php` file:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AppendUserIdToResponse
{
    public function handle(Request $request, Closure $next): Response
    {
        /** @var \Symfony\Component\HttpFoundation\Response $response */
        $response = $next($request);

        $response->headers->set('x-user', $request->user()?->id ?? '-');

        return $response;
    }
}
```

> _Note: I'm taking `$request->user()?->id` here but nothing is stopping you from using `$request->user()?->username` if that makes more sense for you._

We always add a header but if the user is not logged in we default to a `-` which NGINX defaults to when the header is not set at all.

Next we are going to register this new middleware in the `app/Http/Kernel.php` in the `$middleware` array:

```diff
    /**
     * The application's global HTTP middleware stack.
     *
     * These middleware are run during every request to your application.
     *
     * @var array
     */
    protected $middleware = [
        // \App\Http\Middleware\TrustHosts::class,
        \App\Http\Middleware\TrustProxies::class,
        \Fruitcake\Cors\HandleCors::class,
        \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
+       \App\Http\Middleware\AppendUserIdToResponse::class,
    ];
```

> _Note: We are adding it to the global middleware stack to make sure it runs for every request instead of needing to register it per route._

This is it for the Laravel side (don't forget to deploy)!

## Adding the user ID to the NGINX log format

This part is a bit more "custom" and depends on how you have set up your NGINX and log formats, but let's give it a shot!

Custom log formats can be defined in `/etc/nginx/nginx.conf` (within the `http {}` block) and could look something like this:

```nginx
log_format main_ext '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    '"$host" sn="$server_name" '
                    'rt=$request_time '
                    'ua="$upstream_addr" us="$upstream_status" '
                    'ut="$upstream_response_time" ul="$upstream_response_length" '
                    'cs=$upstream_cache_status '
                    'uid="$upstream_http_x_smls_user"';
```

> _Note: this `log_format` is the log format required by [NGINX Amplify](https://amplify.nginx.com/) and used as an example._

To add our user ID to this we would do this:

```diff
log_format main_ext '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    '"$host" sn="$server_name" '
                    'rt=$request_time '
                    'ua="$upstream_addr" us="$upstream_status" '
                    'ut="$upstream_response_time" ul="$upstream_response_length" '
-                   'cs=$upstream_cache_status';
+                   'cs=$upstream_cache_status '
+                   'uid="$upstream_http_x_user"';
```

Notice the added `uid="$upstream_http_x_user"` at the end, this will render as `uid="1"` if user `1` is logged in or `uid="-"` if no user is logged in.

It's possible you do not have a custom `log_format` then your are using the default `combined` format, add this log format which is `combined` with the `uid` field added:

```nginx
log_format combined_with_user_id '$remote_addr - $remote_user [$time_local] '
                                 '"$request" $status $body_bytes_sent '
                                 '"$http_referer" "$http_user_agent" '
                                 'uid="$upstream_http_x_user"';
```

After this we need to find the `access_log` entries in our vhosts and make sure the correct log format is used:

```nginx
server {
    ...
    access_log /var/log/nginx/access.log combined_with_user_id; # or whatever you named your log format
    error_log /var/log/nginx/error.log warn;
    ...
}
```

There is one last optional step, and that is to hide this header from the response. It should not be a security issue but there is no reason to have that header in the response so we are going to hide it:

Locate the `fastcgi_pass` section in your vhost config and add `fastcgi_hide_header x-user;`, could look something like this:

```nginx
location = /index.php {
    include                           /etc/nginx/fastcgi_params;

    fastcgi_param HTTPS               on;
    fastcgi_param HTTP_SCHEME         https;
    fastcgi_param SCRIPT_FILENAME     $realpath_root$fastcgi_script_name;

    fastcgi_pass                      php80;
    fastcgi_index                     index.php;
    fastcgi_hide_header               x-user;
    fastcgi_hide_header               x-powered-by;
    fastcgi_split_path_info           ^(.+\.php)(.*)$;
}
```

## Wrapping up

After you've made these changes NGINX should add `uid="1"` if user `1` is logged in or `uid="-"` if no user is logged in to the access log entries.

One caveat is that this only works for "dynamic" requests, requests that are handled by Laravel/PHP, so static files (like CSS / JS) will always have `uid="-"`.

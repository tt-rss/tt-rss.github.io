---
layout: default
title: Securing Cache Directories
nav_order: 49
---

> [!NOTE]
> Official container images restrict `/cache` access by default. This page applies only to
> legacy host installations.

While nothing critical is stored in cache directories by tt-rss nor do files
have easily guessable names, you may consider forbidding external access over
HTTP to these directories anyway. This is not required, however.

You may also consider restricting access to <code>config.php</code>, just in case.

## Using nginx

```nginx
location /tt-rss/cache {
    deny all;
}

location = /tt-rss/config.php {
    deny all;
}
```

Note: official docker setup has this out of the box.

## Using apache (2.4-syntax)

```apache
<Directory /var/www/html/tt-rss/cache>
    Require all denied
</Directory>

<Directory /var/www/html/tt-rss>
    <Files "config.php">
        Require all denied
    </Files>
</Directory>
```

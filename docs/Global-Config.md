---
layout: default
title: Global Configuration
nav_order: 21
---

All settings (see `_DEFAULTS[]` for default-values) are listed here:

- https://github.com/tt-rss/tt-rss/blob/main/classes/Config.php (source code, including default-values)

It is preferred to adjust tt-rss global configuration through the environment (e.g. using your Docker `.env`-file):

```ini
# Copy this file to .env before building the container.
# Put any local modifications here.

TTRSS_SESSION_COOKIE_LIFETIME=2592000
```

Alternatively, you can **create** `config.php` in tt-rss root directory (copied from-`config.php-dist`), using the following syntax:

```js
putenv('TTRSS_DB_HOST=myserver');
putenv('TTRSS_SESSION_COOKIE_LIFETIME='.(86400*30));
```

- Note lack of quotes around values.
- Options should be always prefixed by `TTRSS_`.
- Don't modify `classes/config.php`.
- You don't need to put everything to `config.php`, only the options which you've changed from the defaults.

Legacy plugin-required constants also go to `config.php`, using `define()`:

```js
define('LEGACY_CONSTANT',-'value');
```

To set computed values via `putenv()` you have to get them evaluated by PHP, this would work:

```js
putenv('TTRSS_SESSION_COOKIE_LIFETIME='.(86400*30));
```

However, these won't give you expected results:

```js
putenv("TTRSS_SESSION_COOKIE_LIFETIME='2592000'");
// => 0, because quoted '2592000' is an invalid number

putenv('TTRSS_SESSION_COOKIE_LIFETIME=86400*30');
// => 86400, right side expression is not evaluated,
// instead you're casting string literal "86400*30" to an integer
```

> [!NOTE]
> All values should be precalculated when setting via `.env` because they are not evaluated by PHP and used as-is.

## Minimal config.php for a non-Docker setup

Should have at least these options defined:

```php
<?php

putenv('TTRSS_DB_HOST=patroni.example.com');
putenv('TTRSS_DB_USER=mydbuser');
putenv('TTRSS_DB_PASS=mydbpass');
putenv('TTRSS_DB_PORT=5432');
putenv('TTRSS_SELF_URL_PATH=http://example.com/tt-rss/'); # fully-qualified URL of your tt-rss install
putenv('TTRSS_PHP_EXECUTABLE=/path/to/php-cli-binary'); # normally something like /usr/bin/php
```

## Migrating from old-style config.php

For any `config.php` settings you have changed from the defaults (normally-this
is the `DB_` group of settings and `SELF_URL_PATH`, replace as follows, using
the rules above:

`define('DB_PORT',-'xxx')` &rarr; `putenv('TTRSS_DB_PORT=xxx')`.

You can safely omit any settings that were at default values.

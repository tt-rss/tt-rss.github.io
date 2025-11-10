---
layout: default
title: FAQ
nav_order: 3
---

{: .note }
> [Docker-related stuff is on a separate page](Installation-Guide#faq)

### I want to check how tt-rss renders my feed / the feed I'm trying to use is parsed incorrectly

tt-rss expects valid XML feed data which is parsed using libxml. Any XML parse errors, should you feel that libxml is misbehaving (which is unlikely), should be reported to libxml developers. We don't add hacks for invalid XML on tt-rss side.

Plugins may affect parsing, consider disabling any plugins before investigating XML-related issues.

### I managed to lock myself out of tt-rss

This assumes you can't simply reset your password via email (login form - forgot my password).

If you have OTP (2FA) enabled and know your password but can't provide an OTP token, you can disable OTP via SQL:

```sql
UPDATE ttrss_users SET otp_enabled = false WHERE login = 'you'
```

If you don't remember your password run the following query:

```sql
UPDATE ttrss_users
    SET pwd_hash = 'SHA1:5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8', salt = '', otp_enabled = false
    WHERE login = 'you'
```

This sets your password back to default (``password``) and disables OTP.

### I have HTTP authentication enabled and get “Your access level is insufficient to run this script” error on login

The problem is that if you have `auth_remote` enabled in [PLUGINS](Global-Config) tt-rss tries to automatically log you in as the user specified by the server using HTTP authentication, which may not have administrative privileges.

The easiest way is simply updating database using CLI (`php ./update.php --update-schema`). Docker setup does this on startup.

Alternatively, you can either temporarily disable `auth_remote` (replace it with `auth_internal`), temporarily disable HTTP authentication, or give yourself administrative permissions using SQL:

```sql
    update ttrss_users set access_level = 10 where login = 'you';
```

### UI is missing CSS or is otherwise visibly broken

- Try opening tt-rss using safe mode, clean browser profile, or an incognito
  window (a different browser would also work)
- Unless you're using SSL, try a different network connection in case your ISP is MITMing you
- Some values of ``Content-Security-Policy`` header may break tt-rss, if you
  have this header set in your httpd config, try disabling it temporarily
- Check for any files in ``(tt-rss)/themes`` not known to Git (especially
  ``default.php`` or ``default.css`` - formerly default CSS themes for tt-rss,
  now removed), try temporarily removing all third party themes from
  ``themes.local``

### Third party theme or plugin broke after update making the UI unusable

Log in to tt-rss in safe mode (use an incognito window if you can't get to login page).

### I want to limit height of images to something more manageable

For combined mode:

```css
body.ttrss_main .cdm .content img, body.ttrss_main .cdm .content video {
    max-height : 90vh;
    height : auto;
}
```

`90vh` means "90% of viewport height". This works on Chromium and derivatives, you can use `90%` for Firefox.

### Feeds stop updating for users who rarely login

This is controlled by a global configuration setting. You can override (or disable) it through environment or `config.php` by setting `TTRSS_DAEMON_UPDATE_LOGIN_LIMIT` to `0`.

Note that this also effectively disables purging of articles stored for inactive users.

### I have questions about article purging / I don't think purging works

Purging is performed on successful feed update, no updates = no purging.

Starred articles are never purged, unread articles are purged if relevant preference is enabled.

Purging is done based on import timestamp, internal to tt-rss. It may be different from article date specified by the feed (i.e. article says it was published on 1970/01/01 but it was imported today). You can see import timestamp if you hover over date in tt-rss web UI.

Import date is bumped every time article is encountered in the feed, otherwise it will get purged and reimported again on every feed refresh, creating duplicates.

When in doubt, use Feed debugger (`f D` on a feed) to see additional purging-related information:

```text
[11:08:10/6783] purging feed...
[11:08:10/6783] purge_feed: interval 60 days for feed XXX, owner: X, purge unread: 1
[11:08:10/6783] purge_feed: deleted 1 articles.
[11:08:10/6783] update done.
```

Related question:

### I archive or delete articles manually and I get duplicates. Why?

Because the articles are still in the feed XML and get pulled in (again) on next feed update.

See also: [Archived Feed](Archived-Feed.md)

### I have used update daemon before, but switched away from it. However, the UI keeps nagging me about the daemon not running or not updating feeds or whatever

Find and delete daemon lock file in <code>LOCK_DIRECTORY</code>. Usually, it's <code>lock/update_daemon.lock</code>. You can also remove <code>update_daemon.stamp</code>.

### I need an URL I can call to subscribe to feed to integrate with some third party browser extension/application

```text
https://example.com/tt-rss/public.php?op=bookmarklets--subscribe&feed_url=%s
```

If feed URL is empty (or not given) tt-rss will display feed subscription dialog.

### I need to get the number of unread articles for specific user

There's a simple unauthenticated endpoint to do just that:

```bash
$ curl -s "https://example.com/tt-rss/public.php?op=getUnread&login=you&fresh=1" ; echo
8;1
$ curl -s "https://example.com/tt-rss/public.php?op=getUnread&login=you" ; echo
8
```

In single user mode, use “admin” for login.

If optional parameter `&fresh=1` is passed via query string, return value includes fresh article count as a `;`-separated string.

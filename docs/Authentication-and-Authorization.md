---
layout: default
title: Authentication and Authorization
parent: Security
---

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

### Authentication Plugins

The bundled authentication plugins are:

* [`auth_internal`](https://github.com/tt-rss/tt-rss/blob/main/plugins/auth_internal/init.php)
  * The default authentication plugin.
  * Utilizes the tt-rss database.
* [`auth_remote`](https://github.com/tt-rss/tt-rss/blob/main/plugins/auth_remote/init.php)
  * Disabled by default.
  * Relies upon external/remote sources to provide identity information.
    * See the plugin source code for the list of sources checked.
    * For backwards compatibility the `Remote-User` request header is included as a trusted source,
      but only checked after certain server-set values.  If you don't want that header to be checked
      you should set `TTRSS_AUTH_REMOTE_TRUSTED_SOURCES` or remove the header before it reaches tt-rss.
      Thank you to EZO Labs Security (GitHub user superbot-helper; https://ezolabs.net, tony@ezolabs.net)
      for highlighting the potential `Remote-User` concern.

### Authorization

* _TODO_

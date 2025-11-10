---
layout: default
title: Plugins
nav_order: 10
---

Tiny Tiny RSS supports many kinds of plugins: social plugins which share
articles to various sites, article filter plugins which mangle feed-provided
data on import (for example, inlining images or extracting full article text
using Readability), hotkey plugins which alter the way keyboard shortcuts work,
etc.

There are two kinds of plugins: user and system. User plugins are enabled in
`Preferences` &rarr; `Plugins`. System plugins require adding them to a [global
configuration](Global-Config) directive <code>PLUGINS</code> which is a
comma-separated list of enabled system plugins, i.e.

```js
putenv('TTRSS_PLUGINS=auth_internal, other_plugin');
```

System plugins are always enabled for all users. If multiple search plugins are loaded, only the first one is used

If you are interested in making plugins, see [Making-Plugins](Making-Plugins),
<https://github.com/topics/tt-rss-plugin>,
<https://github.com/topics/ttrss-plugins>, etc.

### Installing plugins

{: .note }
> First party plugins can be added via built-in plugin installer in `Preferences` &rarr; `Plugins`.

Copy plugin folder to ```tt-rss/plugins.local``` then activate it in the settings panel.
Plugin folder name should correspond to plugin class name defined in ``(plugin)/init.php``,
i.e. ``Af_ExamplePlugin`` should be copied to ``plugins.local/af_exampleplugin``.

## First party plugins (maintained on this site but not bundled with-tt-rss)

<https://github.com/orgs/tt-rss/repositories?q=tt-rss-plugin+sort%3Aname-asc>

## Third party plugins

{: .warning }
> We're not responsible for third party plugins. Use at your own risk.
>
{: .note }
> Third party plugins may be unmaintained and incompatible with newer tt-rss
> code (especially those from the old forums). Please report plugin-related
> problems to their developers.

### Sharing plugins

#### A Tiny Tiny RSS plugin to post to a Wallabag v2 instance

<https://github.com/joshp23/ttrss-to-wallabag-v2>

#### A plugin for Tiny Tiny RSS, to shorten urls via Yourls

<https://github.com/joshp23/tt-rss-yourls>

#### Adds support for sharing links with Shaarli to tt-rss

<https://github.com/joshp23/tt-rss-shaarli>

#### Convert DOI and other links to Sci-Hub links in TT-Rss

<https://github.com/joshp23/ttrss-to-Sci-Hub>

### Feed data manipulation plugins

#### Enable embedded videos in feeds - videoframes

<https://github.com/tribut/ttrss-videoframes>

#### Configurable plugin to replace article stub with content from the linked URL's page

<https://github.com/feediron/ttrss_plugin-feediron>

#### A simple plugin to assist in the display of images from NASA's Astronomy Picture of the Day feed in tt-rss

<https://github.com/joshp23/TTRSS-APOD-Fix>

### Webcomics plugins

#### Comic plugin GU Comics, Married to the sea & Toothpaste for dinner

<https://github.com/tribut/ttrss-comics>

#### Lint/tidy plugin to repair invalid feeds

<https://github.com/Churten/tt-rss-ff-xmllint>

#### Embed content from Tapastic rss streams

<https://github.com/ldidry/af_tapastic.git>

### API plugins

#### FreshRSS / Google Reader API Support

Use any RSS app or client that supports FreshRSS or the Google Reader API.

<https://github.com/eric-pierce/freshapi>

#### Fever API emulator

Simulates the Fever API for reading RSS Feeds with your Fever clients.

<https://github.com/DigitalDJ/tinytinyrss-fever-plugin>

### Other plugins

#### Generate QR codes from article links, with xhr support and no disk cache

<https://github.com/GregThib/ttrss-qrcodegen>

#### Send XMPP notifications via Prosody mod_post_msg

<https://github.com/joshp23/ttrss-notify-xmpp-prosody>

#### Plugins for alternative navigation and night mode

Set of plugins to (1) use cursor keys for a tree-style article navigation; (2) change to a minimal set of hotkeys; (3) toggle night mode for custom themes; (4) change the sort order of unread articles to Oldest first.

<https://github.com/ltGuillaume/FeedMei/tree/master/plugins.local/>

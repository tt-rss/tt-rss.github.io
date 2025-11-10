---
layout: default
title: Making Plugins
parent: Plugins
nav_order: 1
---

Plugins may render new preference panes or embed themselves into several
existing one, store data using simple key/value data or directly in
the database, modify how articles are rendered, alter feed data, and
much more.

You can use sample plugins bundled with tt-rss and [other
plugins](Plugins) as a starting point. Ask on the forums if you need help
with anything specific.

Some useful information may be found here:

- <https://github.com/tt-rss/tt-rss/blob/main/classes/PluginHost.php>
- <https://github.com/tt-rss/tt-rss/blob/main/classes/Plugin.php>

Frontend (JS) uses different hooks, which are defined in [PluginHost.js](https://github.com/tt-rss/tt-rss/blob/main/js/PluginHost.js)

## Localization support

See ``time_to_read`` plugin for [a complete example](https://github.com/tt-rss/tt-rss-plugin-time-to-read)

### Implementation

- Plugin translations are placed in a separate Gettext domain (name equals lowercase plugin-class).
- Translation (.po) file in ``(plugin-dir)/locale/(LANG)/LC_MESSAGES/`` name should correspond to Gettext domain name.

### Using gettext

- On the PHP side, either use helper methods defined in ``classes/plugin.php``
  (base class for all plugins) or call ``_dgettext`` group of functions
  directly.
- On the Javascript side, all translations are merged so you can use the usual
  ``__()`` shortcut function.


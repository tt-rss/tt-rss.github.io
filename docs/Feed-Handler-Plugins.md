---
layout: default
title: Feed Handler Plugins
parent: Plugins
nav_order: 2
---

{: .warning }
> Unless you have a strong need for these plugins, using them is not recommended. If
> you do use them, at least don't enable them for all feeds.

Some plugins utilize global per-feed content hooks which either modify fetched
feed XML data (i.e. fixing broken XML) or even generate it entirely, for tt-rss
to process, for websites that don't actually provide RSS feeds.

While having this ability available for plugins is valuable there are several
downsides:

1. This entirely bypasses rate limiting and article duplicate checking done
   during normal feed update process. A badly written plugin using
   <code>HOOK_FETCH_FEED</code> may cause your tt-rss feed to keep updating
   feeds indefinitely, processing all articles every time, causing unnecessary
   CPU load on your server.

2. If you have full text content plugins enabled, this may cause excessive load
   on origin server, because your tt-rss instance is going to scrape site
   content for every provided article on every feed update.

This may result in your tt-rss instance being banned by origin servers for
causing excessive load and/or your VDS being suspended for suspected bad
behavior.

Additionally, tt-rss being blacklisted by feed publishers may negatively affect
other users.

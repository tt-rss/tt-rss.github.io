---
layout: default
title: Email Digests
nav_order: 42
---

Users may opt into receiving daily (sent once every 24 hours) digests of unread headlines via email.

Digests are sent out by the update daemon or if running <code>update.php —feeds</code>
manually. At most 15 messages are sent in one batch.

{: .note }
> [PHPMailer plugin](https://github.com/tt-rss/tt-rss-plugin-mailer-smtp) is required to send mail under Docker.


* Digests may be customized by editing ``templates/digest_template*.txt``.
* You can preview your digest contents in preferences.


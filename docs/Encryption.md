---
layout: default
title: Encryption
parent: Security
---

Transparent at-rest encryption is optionally supported for sensitive data stored in the database,
currently limited to stored session data and passwords for feeds with authentication enabled.

To enable, [global configuration](Global-Config) option `TTRSS_ENCRYPTION_KEY` should be set to a 32-byte hex string of random bytes,
which may be generated using CLI like this:

```sh
php ./update.php --gen-encryption-key
```

If enabled, existing plaintext login sessions are automatically encrypted when used, plaintext feed passwords are encrypted on feed update.

{: .warning }
> Automatic encryption of plaintext data is a one-way process. If you decide to disable `TTRSS_ENCRYPTION_KEY` afterwards, all encrypted sessions would
> become invalid and you will get logged out. Feed passwords would become unreadable until you either enable encryption back using same key or edit feeds
> manually.

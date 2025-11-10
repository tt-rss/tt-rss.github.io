---
layout: default
title: SSL Certificate Authentication
nav_order: 52
---

{: .warning }
> This guide is considered legacy and is no longer supported as it is not compatible with the
> [stock Docker Compose](Installation-Guide) setup. Please don't report any issues when
> trying to DIY this.

This article details the steps to enable user authentication with tt-rss using a client certificate.

## Prerequisites

You **must** have a working tt-rss installation with SSL. This guide is not intended to walk you through installing tt-rss, nor is it intended to help you enable HTTPS on your web server.

If you have no idea how certificates work (i.e. the terms x509 and PKI make no sense to you), stop now.

This guide includes steps for Nginx. Of course other web servers (e.g.-Apache) support client certificates so you're welcome to use them if you prefer, the steps just aren't included here (but might be added at some point).

This guide was written with Debian 9 in mind, other distros will vary.

## Getting Started

Client certificates are typically created/issued by a private Certificate Authority
(i.e. **you** as the administrator would create the certificates for your users). You
create a root certificate authority and install the **public** certificate for it on
your web server. You then create certificates for each client, signed by the root
certificate authority's private key. Each client is issued their certificate and
private key (often as a single file with a `.p12`-extension).

Note:

1. These certificates are distinct from the ones used for encrypted/HTTPS connections on your server. Those are usually issued by public Certificate Authorities (e.g. Let's Encrypt).
2. Do **not** do the certificate creation on your public, Internet-facing web server. This should be done on a computer that's offline and the certificate authority's private key should be kept in a safe place.

## Server Setup

Install your certificate authority public certificate on the server. The location doesn't really matter but this is how Debian does it:

```sh
sudo cp ttrss-ca.crt /usr/local/share/ca-certificates/
sudo chown root:root /usr/local/share/ca-certificates/ttrss-ca.crt
sudo chmod 644 /usr/local/share/ca-certificates/ttrss-ca.crt
```

(In Debian run the command `sudo update-ca-certificates` to rebuild the list of certificate-authorities.)

Edit your Nginx conf file for your tt-rss installation to add the certificate and have Nginx validate clients with it.

Note:

1. The file path and extension to the certificate are different than above because Debian builds a consolidated list of certificate authorities provided by the vendor, user, etc.; this is why we ran the update-ca-certificates command above and your distro may vary its approach so be aware of that.
2. We use `ssl_verify_client on;` which prevents access to the site unless a valid certificate is provided. If you want **both** password and certificate authentication, use `ssl_verify_client optional;` instead.
3. In the php location we do not use `$ssl_client_v_start` and `$ssl_client_v_end` Nginx variables by default. These variables were added in Nginx version 1.11.7 and Debian 9 does not ship that version. It's included as a comment for future support.

```nginx
server {

    ssl_client_certificate /etc/ssl/certs/ttrss-ca.pem
    ssl_verify_client on;
    # ssl_verify_client optional;

    location ~ [^/]\.php(/|$) {

        fastcgi_param  SSL_CLIENT_M_SERIAL  $ssl_client_serial;
        fastcgi_param  SSL_CLIENT_S_DN      $ssl_client_s_dn;
        # fastcgi_param  SSL_CLIENT_V_START   $ssl_client_v_start;
        # fastcgi_param  SSL_CLIENT_V_END     $ssl_client_v_end;
        fastcgi_param  SSL_CLIENT_V_START   0;
        fastcgi_param  SSL_CLIENT_V_END     0;

    }

}
```

Now restart Nginx:

```sh
sudo systemctl restart nginx
```

Update the tt-rss config file to add `auth_remote` to the `PLUGINS` constant (near the end of the file):

```php
define('PLUGINS', 'auth_internal, auth_remote,-note');
```

You might be tempted to remove `auth_internal` but we still need it so don't.

Next, make sure your client certificate (the `.p12`-file) is installed on your
computer. Different operating systems and browsers do this differently, so you're
pretty much on your own there. However, if you double-click the .p12 file from the
desktop, the operating system should offer to install it for you.

Vendor-provided browsers (e.g. IE/Edge, Safari, etc.) will typically use
certificates provided by the operating system. Third-party installed browsers (e.g.
Firefox) often need to have the .p12 file added to them independently of the
operating system.

After successfully installing the client certificate, open a new browser window/tab and visit your tt-rss install. You should immediately be asked to confirm or select the client certificate you want to use. Select the appropriate one. You may have to login with your username/password; this is expected.

Go to Preferences and scroll to the bottom. Under *Login with an SSL certificate* the *Register* button should now be available. Click it, then *Save configuration*.

At this point you should be able to test if this all works. Logout of your tt-rss
session, clear your browser cache and cookies, then open a new window/tab and visit
your tt-rss install. You may be asked to verify the client certificate (some
browsers ask every session and others remember your choice). Once you select the
certificate it should just log you without using the username/password form.

Finally, if you're never going to use password authentication you could remove
`auth_internal` plugin in `config.php`, just remember to add it back if you remove
certificate support in the future otherwise you'll get the login form but will never
be able to login. You'll also need to enable it if you have to change client
certificates as there will be no other way of logging in.

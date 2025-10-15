---
layout: default
title: Android Client
nav_order: 40
---

## **IMPORTANT**

* The original tt-rss-android project, hosted at https://tt-rss.org/ and its various subdomains,
  [will be gone after 2025-11-01](https://community.tt-rss.org/t/the-end-of-tt-rss-org/7164).
* There are currently no plans to continue maintaining the app.
* The app's source code may be found at https://github.com/tt-rss/tt-rss-android .
* **APK downloads from https://gitlab.tt-rss.org/tt-rss/tt-rss-android/-/releases are no longer working.  This is likely permanent.**
* This page's content should only be treated as a historical reference.
* Refer to https://github.com/tt-rss/tt-rss/discussions/29 for more information.

## Screenshots

[<img src="images/tt-rss-android/Screenshot_20250509_135136.webp" width="120" align="left">](images/tt-rss-android/Screenshot_20250509_135136.webp)

[<img src="images/tt-rss-android/Screenshot_20250509_135154.webp" width="120" align="left">](images/tt-rss-android/Screenshot_20250509_135154.webp)

[<img src="images/tt-rss-android/Screenshot_20250509_135217.webp" width="120" align="left">](images/tt-rss-android/Screenshot_20250509_135217.webp)

[<img src="images/tt-rss-android/Screenshot_20250509_135426.webp" width="120" align="left">](images/tt-rss-android/Screenshot_20250509_135426.webp)

[<img src="images/tt-rss-android/Screenshot_20250509_135438.webp" width="120">](images/tt-rss-android/Screenshot_20250509_135438.webp)

## Download

Releases are built automatically for every commit pushed to master branch and [published on Gitlab](https://gitlab.tt-rss.org/tt-rss/tt-rss-android/-/releases).

> [!WARNING]
> This is the only official and supported way to get the APK. If you see the app on any marketplace or anywhere else, it had been uploaded there without my knowledge or consent.

Release APKs are signed. You can verify the signature using [apksigner](https://developer.android.com/tools/apksigner):

```sh
$ apksigner verify --print-certs org.fox.ttrss-signed.apk
Signer #1 certificate DN: CN=Andrew Dolgov, OU=N/A, O=tt-rss.org, L=Saint-Petersburg, ST=N/A, C=RU
Signer #1 certificate SHA-256 digest: c74664ba0fd8f8c97e2a548926609df1369236dd9d9d14c0e5c20b8c2b08cf06
Signer #1 certificate SHA-1 digest: ac97a3ced638cd750272dab50c08ca979910dc74
Signer #1 certificate MD5 digest: e4f38ce99c44714e3c21821a1c13717f
```

Signature digests must match above values.

## FAQ

### How do I automatically update the APK?

There's no built-in auto updater, I suggest using [Obtainium](https://obtainium.imranr.dev/) instead.

When adding new application in Obtainium, paste [app project page](https://gitlab.tt-rss.org/tt-rss/tt-rss-android) in first field, and select "Gitlab" in the "Override source" dropdown below.

### I want to help test the app! Are there development builds available?

Sometimes. Releases are not created for signed non-master branch builds, you can get the APKs from build pipeline artifacts [here](https://gitlab.tt-rss.org/tt-rss/tt-rss-android/-/jobs) - look for `build-signed-branch-apk`. Non-master builds install as a separate application with a gray icon.

### Why are you not on Google Play?

As a small time FOSS developer, dealing with Google and their marketplace is a tremendous pain in the ass.

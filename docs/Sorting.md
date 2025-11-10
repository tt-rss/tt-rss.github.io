---
layout: default
title: Sorting
parent: Features
---

Tiny Tiny RSS provides four options for how articles appear within a
selected feed: `Default`, `Newest`, `Oldest`, `Title`.

{: .note }
> Special feeds (e.g. *Starred articles*) have unique sorting when *Default* is selected, otherwise they behave as described below.

- *Descending score* means **higher numbers** are shown before lower numbers.
- *Descending date/time* means **more recent** is shown before less recent.
- *Ascending date/time* means **less recent** is shown before more recent.

### Default

This is the default (surprise!) and is recommended.

1. Descending [score](Scoring).
2. Descending date/time the article was added into the Tiny Tiny RSS database.
3. Descending date/time the feed's site states the article was published or changed.

### Newest

1. Descending date/time the feed's site states the article was published or changed.

### Oldest

1. Ascending date/time the feed's site states the article was published or changed.

### Title

1. Alphabetically by the title of the article.
2. Ascending date/time the article was added into the Tiny Tiny RSS database
3. Ascending date/time the feed's site states the article was published or changed.

## Special Feeds

When *Default* is selected these special feeds behave as described below.

### Starred articles

1. Descending date/time when the article was starred.
2. Descending date/time the article was added into the Tiny Tiny RSS database.
3. Descending date/time the feed's site states the article was published or changed.

### Published articles

1. Descending date/time when the article was published.
2. Descending date/time the article was added into the Tiny Tiny RSS database.
3. Descending date/time the feed's site states the article was published or changed.

### Recently read

1. Descending date/time when the article was marked as read in Tiny Tiny RSS.

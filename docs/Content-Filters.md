---
layout: default
title: Content Filters
parent: Features
---

Filters are a very powerful and flexible tool which may significantly ease the
task of extracting useful information from the sea of data that is RSS feeds.
Filters are applied to articles based on [regular
expression](http://en.wikipedia.org/wiki/Regular_expression) matches against
specified fields. After the match had been found, configured actions are taken.
Matching is case-insensitive,
[PCRE](http://php.net/manual/en/reference.pcre.pattern.syntax.php) pattern
syntax is used.

{: .note }
> Filter test dialog may not give entirely accurate results, especially for
> complex filters. It is suggested to test filters using ``Feed debugger`` (hotkey `f-D`) if you
> feel that some filter is somehow misfiring on a specific feed.

### Load order

Filters are loaded in user-specified order and applied sequentially. It is
possible to reorder filters using drag and drop. If no manual sorting is
specified, filters are sorted alphabetically according to user configured
caption. If no caption is specified for any filter, loading order is not
guaranteed.

### Filter objects

Each filter object may contain an arbitrary amount of regular expression rules
and actions. Each expression may have inverse flag set, which inverts matching
result. On top of that, filter may also have an inverse flag, which inverts the
final matching.

- Filter object may be configured to successfully match when either one or
all rules match.
- Regular expressions may be matched against several article fields, such
as, title, content, author, etc.
- Do not include delimiters (e.g.-<code>/</code>) when writing regular
expressions.

### Matching articles and applying actions

Filter matching is performed during feed processing.

{: .note }
> Some actions may be applied only when the article is initially imported from the
> feed. Other actions may be applied every time article is seen in the originating
> feed. It is suggested to only rely on filters applying to articles imported
> after the filter had been created - they will not retroactively apply to your
> article database.

Several actions are available:

{: .warning }
> Filters may not apply actions conditionally based on previous filters. All actions for an article are applied together, once.

1. ``Delete article`` - do not import article from the feed, does not
actually delete anything from the database
2. ``Mark as read`` - imports article automatically marked as read
3. ``Set starred`` - sets article starred automatically on import
4. ``Assign tags`` - assigns a comma-separated list of custom tags on import
5. ``Publish article`` - sets article published automatically on import
6.-``Modify score`` ([Scoring](Scoring.md)) - modifies article overall score based on
the parameter, a signed integer number. Final article score is calculated after all filters had been applied and is a sum of all matched scoring actions.
7. ``Assign label`` - assigns specified label to the article on import
8. ``Stop / Do nothing`` - stops further filter processing for this article, no following filters will be checked nor rules applied.
9. ``Invoke plugin`` - runs a plugin action when filter is matched
10. ``Ignore tags`` - a comma-separated list of tags which will be skipped on article import

After all matching filters had been computed for the article, it is either
imported with modifications as specified by the rules, or dropped if `Delete
article` action has been found.

---
layout: default
title: Search
parent: Features
---

{: .note }
> This page's content only applies to the built-in search. Plugins may override the default search behavior.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Introduction

A search query consists of one or more keywords, of which there are 3 types:

* <a id="keyword-text"></a> **Text**
  * A Text keyword may be a single word such as `ocean` or successive words enclosed in quotes such as `"pacific ocean"`.
  * These keywords are searched using the PostgreSQL [Full Text Search](#postgresql-full-text-search) engine,
   which supports [word stemming](#word-stemming) and [logical operators](#logical-operators).
* <a id="keyword-field"></a> **Field**
  * A Field keyword allows filtering articles by supported fields (shown below).
  * Examples:
    * `star:true`, `star:false` - match starred or not starred articles
    * `unread:true`, `unread:false` - match unread or read articles
    * `pub:true`, `pub:false` - match published or unpublished articles
    * `title:sometext`, `title:"two words"` - match articles with a title containing the specified text (sub-string match)
    * `author:sometext`, `author:"two words"` - match articles with an author containing the specified text (sub-string match)
    * `note:true`, `note:false`, `note:sometext`, `note:"two words"` - match articles with any note, no note, or a note containing the specified text (sub-string match)
    * `label:true`, `label:false`, `label:somelabel`, `label:"two words"` - match articles with any label, no label, or having the specified label (exact-string match)
    * `tag:true`, `tag:false`, `tag:sometag`, `tag:"two words"` - match articles with any tag, no tag, or having the specified tag (exact-string match)
* <a id="keyword-date"></a> **Date**
  * A Date keyword allows filtering articles by their publication (or last updated) date.
  * Examples:
    * `@2025-10-28` (formatted as `@YYYY-MM-DD`)
    * `@2025/10/28` (formatted as `@YYYY/MM/DD`)
    * `@28/10/2025` (formatted as `@DD/MM/YYYY`)
    * `@"28 Oct 2025"`
    * `@"October 28"` (of the current year)
    * `@today`
    * `@yesterday`
    * `@"2 days ago"`
    * `@"last Monday"`
  * {: .note }
    > A Date keyword has to represent a fixed day. For example `@"last week"`, `@2023-11` or `@2024` cannot be used because they represent a range of several days.


A keyword starting with `-` (negative sign) represents a negative match. `-` can be applied before any type of keyword.
For example: `-unwanted`, `-"unwanted words"`, `-title:unwanted`, `-tag:"unwanted words"` or `-@yesterday`.

A logical `AND` operator is automatically applied between keywords (and therefore should not be explicitly provided).
For example: `ocean "tree flower" note:true -title:"orange color"` searches for articles containing the word _ocean_ (with [stemming](#word-stemming))
AND containing the phrase _"tree flower"_ (with [stemming](#word-stemming)) AND having any note AND having a title not containing the phrase _"orange color"_.

Other [logical operators](#logical-operators) are only supported around a [Text keyword](#keyword-text), as they're processed by the
PostgreSQL [Full Text Search](#postgresql-full-text-search) engine.  A [Field keyword](#keyword-field) or [Date keyword](#keyword-date)
does not support those PostgreSQL [logical operators](#logical-operators).


## PostgreSQL Full Text Search

Tiny Tiny RSS uses PostgreSQL, which includes a [Full Text Search engine](https://www.postgresql.org/docs/current/textsearch-intro.html){:target="_blank" rel="noreferrer"} (external link).

This search engine supports two main features:
* [Word stemming](#word-stemming)
* [Logical operators](#logical-operators)

### Word stemming

Word stemming is a process to find the stem (root) of a word. For example, in English the words _security_, _secure_ and _secured_ all share the same stem: _secur_.
PostgreSQL names this stem a _lexeme_. A _lexeme_ is a normalized string, where different forms of the same word are made alike.

Word stemming is only available for [Text keywords](#keyword-text).

Here's an example of how word stemming works in relation to Tiny Tiny RSS:
1. Assume an RSS feed provides an article containing the word _security_.  If the user has configured the language of this feed as English,
   then the word _security_ is stored in the database as its lexeme _secur_.
2. Later, the user opens the search form, selects the English language, and searches for _secured_.
3. Tiny Tiny RSS sends _secured_ in a query to PostgreSQL.
4. PostgreSQL converts this query to the lexeme _secur_.  As both lexemes are identical, the article containing _security_ matches the search query _secured_.
5. Tiny Tiny RSS includes the aforementioned article in the list presented to the user.

Word stemming is powerful, but has a notable drawback: both languages of the feed and of the search query have to be well configured.
Indeed, the word stemming process depends on the language.  French and English words, for example, are not stemmed in the same way,
so comparing them may lead to unexpected results.

In Tiny Tiny RSS there is a special language named _Simple_. Word stemming in the _Simple_ language is almost equivalent to exact string matching.
With the _Simple_ language only punctuation such as commas are removed. The power of word stemming isn't applied, but _Simple_ works well when dealing with multiple languages.

It's also possible to search for a word with a specific prefix using the syntax `prefix:*` (e.g. `secu:*` matches every word starting with `secu`).

### Logical operators

A [Text keyword](#keyword-text) can be surrounded by logical operators supported by the PostgreSQL [Full Text Search engine](#postgresql-full-text-search).

{: .note }
> Due to current parser limitations, these logical operators cannot be applied on a [Field keyword](#keyword-field) nor on a [Date keyword](#keyword-date).

PostgreSQL provides:
* `!` : logical NOT
* `&` : logical AND
* `|` : logical OR
* `(` and `)` : parentheses can be used to control nesting of operators. Without parentheses `|` binds least tightly, then `&`, and then (most tightly) `!`.

For example: `ocean & ( ( pacific | atlantic ) & ! "black sea" )`

{: .warning }
> Due to current parser limitations, the handling of spaces is important:
> * Spaces are **required around words enclosed in quotes** such as `"black sea"`, otherwise the parser does not detect the quotes.
> * Spaces are **recommended around single words** such as `atlantic`, otherwise the word is not highlighted (see [highlighting limitations](#highlighting-limitations)).
> * Spaces are **recommended around logical operators**, otherwise highlighting may not work correctly.

{: .warning }
> Due to current parser limitations, when at least one of the aforementioned operators is detected Tiny Tiny RSS does not apply the default _AND_ (`&`) operator,
> and instead expects the whole query to be well formatted.
>
> For example: The query `one two` works as expected because no operator is detected and Tiny Tiny RSS adds an _AND_.  `one two & three`, however, fails because
> Tiny Tiny RSS detects the `&` operator, expects the whole query to be well formatted, and doesn't add the missing _AND_ (`&`) operator between words `one` and `two`.

{: .note }
> When a search query contains [Field keywords](#keyword-field) or [Date keywords](#keyword-date) _and_ [Text keywords](#keyword-text) using logical operators,
> it's recommended to write the [Text keywords](#keyword-text) at the end (or the beginning) and to surround them with parentheses.
> For example: when reading `-title:submarine @yesterday ( pacific | atlantic )` one can easily understand that the parentheses contains a complex fragment
> that has to be well formatted with no missing operator.

{: .note }
> Due to current parser limitations, the `-` negation does not work before a parenthesis (only before a [Text keyword](#keyword-text)).
> When a parentheses group needs to be negated, use the `!` operator. For example: `-title:submarine @yesterday ( ! ( pacific | atlantic ) )`


## Quoting variants

When a [Text keyword](#keyword-text) contains spaces, and is negated, it can be written in two ways:
* `-"pacific ocean"` (recommended)
* `"-pacific ocean"`

When a [Field keyword](#keyword-field) contains spaces, it can be written in two ways:
* `title:"two words"` (recommended)
* `"title:two words"`

A negated [Field keyword](#keyword-field) can be written in two ways:
* `-title:"two words"` (recommended)
* `"-title:two words"`

When a [Date keyword](#keyword-date) contains spaces, it can be written in two ways:
* `@"two words"` (recommended)
* `"@two words"`

A negated [Date keyword](#keyword-date) can be written in two ways:
* `-@"two words"` (recommended)
* `"-@two words"`


## Highlighting limitations

A [Text keyword](#keyword-text) can be a single word such as `ocean`. If the article contains `ocean` or `oceanographer`, the `ocean` fragment is highlighted.

A [Text keyword](#keyword-text) can also be successive words enclosed in quotes such as `"pacific ocean"`. If the article contains `pacific oceanographer`, the `pacific ocean` fragment is highlighted.

{: .note }
> Due to incomplete implementation, the word is not correctly highlighted when a [stemmed](#word-stemming) variant is used.
> For example: if the user searches for _secured_ articles containing _security_ are displayed, however as _secured_ is not in the content it is not highlighted.

{: .note }
> If the searched word is negated using `-` it is not highlighted.
> Due to incomplete implementation a negation done with `!` isn't detected, so words negated in a such way are still highlighted. Use `-` instead, if desired.


## Undetected errors

{: .warning }
> Due to current parser limitations most syntax errors are undetected. When a user enters a badly-formatted search query it is incorrectly parsed,
> no message is displayed, and the results may be unexpected.


## Contributions are welcome

The current search query parser implements basic keyword splitting; it works in most cases, but has the disadvantages presented above.

Desired areas of improvement include:
* logical operators and grouping around any type of keyword
* proper highlighting in all cases
* detection of invalid queries, with a warning displayed

Contributions are welcome!

---
layout: default
title: Search
parent: Features
---

{: .note }
> This only applies to the built-in search. Search plugins may override this syntax.

A search query consists of one or several keywords.

<a id="text-keyword"></a>A keyword can be a **text keyword**. A text keyword is a single word such as `ocean`, or successive words enclosed in quotes such as `"pacific ocean"`. These keywords are searched using the PostgreSQL [Full Text Search](#full-text-search) engine. This engine supports [word stemming](#word-stemming), and [logical operators](#logical-operators).

<a id="namevalue-keyword"></a>A keyword can also be a **name-value keyword**:
- `star:true`, `star:false` - match starred or not starred articles
- `unread:true`, `unread:false` - match unread or read articles
- `pub:true`, `pub:false` - match published or unpublished articles
- `title:sometext`, `title:"two words"` - match articles with a title containing the specified text (sub-string match)
- `author:sometext`, `author:"two words"` - match articles with an author containing the specified text (sub-string match)
- `note:true`, `note:false`, `note:sometext`, `note:"two words"` - match articles with a note, no note, or a note containing the specified text (sub-string match)
- `label:true`, `label:false`, `label:somelabel`, `label:"two words"` - match articles with a label, no label, or belonging to the specified label (exact-string match)
- `tag:true`, `tag:false`, `tag:sometag`, `tag:"two words"` - match articles with a tag, no tag, or associated to the specified tag (exact-string match)

A keyword can also be a **date keyword**:
- `@somedate` - match by article publication [date](#date_keyword)

A keyword starting with `-` (negative sign) is considered a negative match. The `-` can be applied before any type of keyword. For example `-unwanted`, `-"unwanted words"`, `-title:unwanted`, `-tag:"unwanted words"` or `-@yesterday`.

A logical `AND` operator is applied between keywords. For example `ocean "tree flower" note:true -title:"orange color"` searches for articles containing the word _ocean_ (with [stemming](#word-stemming)) AND the phrase _"tree flower"_ (with [stemming](#word-stemming)) AND a note AND a title not containing the string _"orange color"_. This _AND_ must not be written, it is applied by default.

Other [logical operators](#logical-operators) are only supported around a [text keyword](#text-keyword), because they are processed by PostgreSQL [Full Text Search](#full-text-search) engine. A [name-value keyword](#namevalue-keyword) or [date keyword](#date_keyword) does not support those PostgreSQL [logical operators](#logical-operators).


<a id="full-text-search"></a>
## PostgreSQL Full Text Search

Tiny Tiny RSS uses a PostgreSQL database, providing a [Full Text Search engine](https://www.postgresql.org/docs/current/textsearch-intro.html) (external link).

It supports two main features:
- [Word stemming](#word-stemming)
- [Logical operators](#logical-operators)

<a id="word-stemming"></a>
### Word stemming

Word stemming is a process to find the stem (root) of a word. For example, in English the words _security_, _secure_ and _secured_ all share the same stem: _secur_. PostgreSQL names this stem a _lexeme_. A _lexeme_ is a normalized string so that different forms of the same word are made alike.

Word stemming is only available for [text keywords](#text-keyword).

Here is a full example. A RSS feed provides an article containing the word _security_. If the user has configured the language of this feed as English, then this word _security_ is stored in the database as its lexeme _secur_.
Later, the user opens the search form, selects the English language, and searches for _secured_. Tiny Tiny RSS sends _secured_ to PostgreSQL, which converts this query to the lexeme _secur_. As both lexemes are identical, the article containing _security_ matches the search query _secured_.

Word stemming is powerful, but has one drawback: both languages of the feed and of the search query have to be well configured. Indeed, the word stemming process depends on the language: French and English words are not stemmed in the same way, so comparing them may lead to unexpected results.

In Tiny Tiny RSS there is a special language named _Simple_.. Word stemming in the _Simple_ language is almost equivalent to exact string matching. With the _Simple_ language, only punctuation such as commas are removed. The power of word stemming is thus not applied, but it works well in usages with multiple languages.

The user can also manually set a prefix in the search query using the syntax `secu:*` which matches every word starting by `secu`.

<a id="logical-operators"></a>
### Logical operators

A [text keyword](#text-keyword) can be surrounded by logical operators provided by the PostgreSQL [Full Text Search](#full-text-search) engine.

{: .note }
> Due to current parser limitations, these logical operators cannot be applied on a [name-value keyword](#namevalue-keyword) nor on a [date keyword](#date_keyword).

PostgreSQL provides:
- `!` : logical NOT
- `&` : logical AND
- `|` : logical OR
- `(` and `)` : parentheses can be used to control nesting of operators. Without parentheses, `|` binds least tightly, then `&`, and `!` most tightly.

For example: `ocean & ( ( pacific | atlantic ) & ! "black sea" )`

{: .warning }
> Due to current parser limitations, the handling of space is important:
> - Spaces are **required around words enclosed in quotes** such as `"black sea"`, otherwise the parser does not detect the quotes.
> - Spaces are **recommended around single words** such as `atlantic`, otherwise the word is not highlighted (please also see [highlighting limitations](#highlighting_limitations)).
> - Spaces are **recommended around logical operators**, otherwise highlighting may not work correctly.

{: .warning }
> Due to current parser limitations, when at least one operator is detected Tiny Tiny RSS does not apply the default _AND_ operator. Tiny Tiny RSS expects the whole query to be well formatted. For example the query `one two` works because no operator is detected, so Tiny Tiny RSS adds the _AND_.
> However, `one two & three` fails because Tiny Tiny RSS detects the `&` operator, so expects the whole query to be well formatted, and does not add the missing `&` between the words `one` and `two`.

{: .note }
> When a search query contains [name-value](#namevalue-keyword)/[date](#date_keyword) keywords and [text keywords](#text-keyword) using _logical operators_, it is recommended to write the [text keywords](#text-keyword) at the end (or the beginning), and to surround them with parentheses.
> For example when reading `-title:submarine @yesterday ( pacific | atlantic )` one can easily understand that the parentheses contains a complex fragment that has to be well formatted with no missing operator.

{: .note }
> Due to current parser limitations, the `-` negation does not work before a parenthesis (only before a [text keyword](#text-keyword)). When a parentheses group needs to be negated, use the `!` operator. For example: `-title:submarine @yesterday ( ! ( pacific | atlantic ) )`


<a id="date_keyword"></a>
## Date keyword

A _date keyword_ can filter articles based on their publication or update date:
- `@2025-10-28` formatted as `@YYYY-MM-DD`
- `@2025/10/28` formatted as `@YYYY/MM/DD`
- `@28/10/2025` formatted as `@DD/MM/YYYY`
- `@"28 oct 2025"`
- `@"October 28"` (of the current year)
- `@today`
- `@yesterday`
- `@"2 days ago"`
- `@"last monday"`

{: .note }
> A _date keyword_ has to represent a fixed day. For example `@"last week"`, `@2023-11` or `@2024` cannot be used because they represent a range of several days.


<a id="quoting_variants"></a>
## Quoting variants

When a [text keyword](#text-keyword) contains spaces, and is negated, it can be written in two ways:
- `-"pacific ocean"` - the recommended usage because it is more readable
- `"-pacific ocean"`

When a [name-value keyword](#namevalue-keyword) contains spaces, it can be written in two ways:
- `title:"two words"` - the recommended usage because it is more readable
- `"title:two words"`

The negative expression can be written in two ways:
- `-title:"two words"` - the recommended usage because it is more readable
- `"-title:two words"`

When a [date keyword](#date_keyword) contains spaces, it can be written in two ways:
- `@"two words"` - the recommended usage because it is more readable
- `"@two words"`

The negative expression can be written in two ways:
- `-@"two words"` - the recommended usage because it is more readable
- `"-@two words"`

<a id="highlighting_limitations"></a>
## Highlighting limitations

A [text keyword](#text-keyword) can be a single word such as `ocean`. If the article contains `ocean` or `oceanographer`, the `ocean` fragment is highlighted.
A [text keyword](#text-keyword) can also be successive words enclosed in quotes such as `"pacific ocean"`. If the article contains `pacific oceanographer`, the `pacific ocean` fragment is highlighted.

{: .note }
> Due to incomplete implementation, the word is not correctly highlighted when a [stemmed](#word-stemming) variant is used. For example, if the user searches for _secured_, articles containing _security_ are displayed, however as _secured_ is not in the content, it is not highlighted.

If the searched word is prefixed by the negation `-`, it is not highlighted.

{: .note }
> Due to incomplete implementation, the negation with `!` is not detected, so words negated in a such way are still highlighted. Use the `-` sign instead.


<a id="undetected_errors"></a>
## Undetected errors

{: .warning }
> Due to current parser limitations, most syntax errors are undetected. When user enters a badly formatted search query, it is incorrectly parsed, no message is displayed, and the results are unexpected.


<a id="contributions"></a>
## Contributions are welcome

The current search query parser implements a basic keyword splitting. It works in most cases, but has the disadvantages presented above.

It is maintained with best effort until someone volunteers to create a full parser with:
- logical operators and grouping around any type of keyword
- highlighting supported in all cases
- detection of invalid queries, with a warning displayed

Contributions are welcome!

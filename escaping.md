# Escaped Text

The fundamental idea of escaping is to provide an injective (and hence invertible) function from arbitrary strings to strings with some pattern not occurring. This could mean that a set of reserved characters do not occur in the output, or that a certain character does not occur alone.

The [netstring](https://cr.yp.to/proto/netstrings.txt) approach of putting the length before the data is an alternative to escaping. NUL delimited text is another alternative to escaping in cases where text cannot contain `\0` (like C strings).

The rest of this document is a review of some of the ways escaping comes up in computing.


## url/uri encoding

Data inside web addresses must avoid the `/` separator and several other characters, as documunted in the URI RFC. To allow these chars to still be passed to a web application we use percent encoding with hex digits: `%XX`.

The choice of `%` as the escape char is good. Since it's not using the more common `\` URLs can usually be put into a string literal without another layer of escaping.

Originaly in [RFC 2396][rfc 2396] the \ was not allowed inside a URL. It was marked as "unwise". In the latest spec, [RFC 3986][rfc 3986] it is allowed but should be escaped as `%5C`.


## HTML escaping

HTML and XML allow you to mix arbitrary text and `<tags>`. So to include in our text the same [metacharacters][wikipedia metacharacter] that are used to described tags they will need escaped.

The `&` char is used to escape special characters in HTML. They call it a [character reference][html character references]. Characters can be referenced by decimal, hex or by name.

According to the [rules for parsing data inside HTML tags][html data parser] one only really needs to escape `&` and `<`. But it is advised to also escape `>`. The main restriction on html data is that it must not contain the string `"</"`. [Here](https://www.w3.org/TR/html5/syntax.html#restrictions-on-the-contents-of-raw-text-and-escapable-raw-text-elements).


## sql escaping

A lot of websites have been exploited from user data ending up unescaped in part of an SQL query. If a web application forgets to escape something user controlled that ends up they can use things like [delimiter collision][wikipedia delimiter collision] to perform their own SQL queries and take over the site.


## Command line arguments and shell scripting

Command line arguments starting with - are usually understood as flags. The magic `--` flag enables one to pass further arguments which are interpreted literally instead of checked to see if they are flags.

On the ext4 filesystem, file names/paths can contain any byte except 0. In shell scripting it is often useful to use `find -print0` to get NUL delimited list of filenames to loop over, although usually people use newline or as a delimited which is not 100% but works almost all the time.

## base64 encoding

As base64 uses such a small alphabet of plaintext characters any data that has been base64 encoded can basically be put in any context without further escaping. In URLs, in HTML, strings inside programming languages, in email. etc.


## string escaping in programming languages

The primary place escaped text happens is in string literals. Double quotes are used to contain and delimit the text.

The syntax of a string literal, in regex is at a very basic level something like: `/"([^"]|\"|\\)*"/`. [wiki](https://en.wikipedia.org/wiki/String_literal) gets this. It's not `"[^"]*"`.

* [Rust](https://doc.rust-lang.org/1.7.0/reference.html#string-literals) basic. newlines allowed. Newlines + whitespace ahead stripped if preceeded by `\`. `x` `u` and `nrt`. It also has raw string literals. A bit like lua's long ones.
* [Go](https://golang.org/ref/spec#String_literals)
* [C](https://en.wikipedia.org/wiki/C_syntax#Strings)
* [POSIX shell](http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html). Shell supports things like glob `*` and `~` which expand to multiple files or a path. To enter a literal asterisk you need to either escape it `\*`, `\~` or put it in a string.
* [bash](http://wiki.bash-hackers.org/syntax/quoting) has weak and strong quoting, supports escapes much like C.
* [Python](https://docs.python.org/dev/reference/lexical_analysis.html#string-and-bytes-literals) has a few different sorts of string literal.
* [Ruby](https://docs.ruby-lang.org/en/2.0.0/syntax/literals_rdoc.html) - needs # escaped. Also has `%(...)`
* [Lua](https://www.lua.org/manual/5.3/manual.html#3.1) - `\z` is special. `\xXX`, where XX is a sequence of exactly two hexadecimal digits `\u{XXX}` is for unicode. brackets mandatory. The long literal strings `[==[` `]==]` are a unique approach.
* [Javascript](http://www.ecma-international.org/ecma-262/9.0/index.html#sec-literals-string-literals) - Probably the simplest string language: x for hex, u for unicoed. special escape chars '"\bfnrtv. Also supports \0.

Another overview here: [codecodex - escape sequences](http://www.codecodex.com/wiki/Escape_sequences_and_escape_characters).

## regex escaping

`.` is usually a metacharacter representing any char, so if you want to match a literal dot you need `\.`.

A very weird choice by sed is that `(` and `)` are matched literally, and to use parens as grouping you need `\(` `\)`. This is a kind of reverse escaping which makes it difficult to remember. I think it is done for the purpose of brevity, assuming that matching brackets literally is done more often than grouping is done - I find the opposite to be true though.

Regex often falls victim to [leaning toothpick syndrome][wikipedia toothpick syndrome].


## UNIX terminals

UNIX terminals use in-band control . This means just `cat`ting a file can completely mess up your terminal if the file contains these control codes. This is a mixed bag of good and bad. Command line programs should escape all terminal control sequences to avoid in band signalling.


## Gopher

Gopher terminates files with a period `.`. [Spec here][gopher]. Because of this it's a bit of extra work to have text files that start with a period in a gopher text page:

```
Note:  Lines beginning with periods must be prepended with an extra
     period to ensure that the transmission is not terminated early.
     The client should strip extra periods at the beginning of the line.
```

[wikipedia metacharacter]: https://en.wikipedia.org/wiki/Metacharacter
[wikipedia percent-encoding]: https://en.wikipedia.org/wiki/Percent-encoding
[wikipedia escape-character]: https://en.wikipedia.org/wiki/Escape_character
[wikipedia toothpick syndrome]: https://en.wikipedia.org/wiki/Leaning_toothpick_syndrome
[wikipedia delimiter collision]: https://en.wikipedia.org/wiki/String_literal#Delimiter_collision
[wikipedia delimiter collision 2]: https://en.wikipedia.org/wiki/Delimiter#Delimiter_collision
[wikipedia in-band signalling]: https://en.wikipedia.org/wiki/In-band_signaling
[rfc 2396]: https://tools.ietf.org/html/rfc2396
[rfc 3986]: https://tools.ietf.org/html/rfc3986
[html character references]: https://www.w3.org/TR/html5/syntax.html#character-references
[html data parser]: https://www.w3.org/TR/html5/syntax.html#data-state
[gopher]: https://tools.ietf.org/html/rfc1436

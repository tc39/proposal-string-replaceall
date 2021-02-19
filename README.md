# `String.prototype.replaceAll` proposal

## Status

Champion: Mathias Bynens (Google, @mathiasbynens).

This proposal is at stage 4 of [the TC39 process](https://tc39.es/process-document/), and is scheduled to be included in ES2021.

## Motivation

(Also see [our TL;DR explainer](https://v8.dev/features/string-replaceall).)

Currently there is no way to replace all instances of a substring in a string without use of a global regexp.
`String.prototype.replace` only affects the first occurrence when used with a string argument. There is a lot of evidence that developers are trying to do this in JS — see the [StackOverflow question](https://stackoverflow.com/q/1144783/96656) with thousands of votes.

Currently the most common way of achieving this is to use a global regexp.

```js
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.replace(/\+/g, ' ');
```

This approach has the downside of requiring special RegExp characters to be escaped — note the escaped `'+'`.

An alternate solution is to combine `String#split` with `Array#join`:

```js
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.split('+').join(' ');
```

This approach avoids any escaping but comes with the overhead of splitting the string into an array of parts only to glue it back together.

## Proposed solution

We propose the addition of a new method to the String prototype - `replaceAll`. This would give developers a straight-forward way to accomplish this common, basic operation.

```js
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.replaceAll('+', ' ');
```

It also removes the need to escape special regexp characters (note the unescaped `'+'`).

## High-level API

The proposed signature is the same as the existing `String.prototype.replace` method:

```js
String.prototype.replaceAll(searchValue, replaceValue)
```

Per the current TC39 consensus, `String.prototype.replaceAll` behaves identically to `String.prototype.replace` in all cases, **except** for the following two cases:

1. If `searchValue` is a string, `String.prototype.replace` only replaces a single occurrence of the `searchValue`, whereas `String.prototype.replaceAll` replaces *all* occurrences of the `searchValue` (as if `.split(searchValue).join(replaceValue)` or a global & properly-escaped regular expression had been used).
1. If `searchValue` is a non-global regular expression, `String.prototype.replace` replaces a single match, whereas `String.prototype.replaceAll` throws an exception. This is done to avoid the inherent confusion between the lack of a global flag (which implies "do NOT replace all") and the name of the method being called (which strongly suggests "replace all").

Notably, `String.prototype.replaceAll` behaves just like `String.prototype.replace` if `searchValue` is a global regular expression.

## Comparison to other languages

* Java has [`replace`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#replace-java.lang.CharSequence-java.lang.CharSequence-), accepting a `CharSequence` and replacing all occurrences. Java also has [`replaceAll`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#replaceAll-java.lang.String-java.lang.String-) which accepts a regex as the search term (requiring the user to escape special regex characters), again replacing all occurrences by default.
* Python [`replace`](https://www.tutorialspoint.com/python/string_replace.htm) replaces all occurrences, but accepts an optional param to limit the number of replacements.
* PHP has [`str_replace`](http://php.net/manual/en/function.str-replace.php) which has an optional limit parameter like python.
* Ruby has [`gsub`](https://ruby-doc.org/core/String.html#method-i-gsub), accepting a regexp or string, but also accepts a callback block and a hash of match -> replacement pairs.

## FAQ

### What are the main benefits?

A simplified API for this common use-case that does not require RegExp knowledge. A way to global-replace strings without having to escape RegExp syntax characters. Possibly improved optimization potential on the VM side.

### What about adding a `limit` parameter to `replace` instead?

A: This is an awkward interface — because the default limit is 1, the user would have to know how many occurrences already exist, or use something like Infinity.

### What happens if `searchValue` is the empty string?

`String.prototype.replaceAll` follows the precedent set by `String.prototype.replace`, and returns the input string with the replacement value spliced in between every UCS-2/UTF-16 code unit.

```js
'x'.replace('', '_');
// → '_x'
'xxx'.replace(/(?:)/g, '_');
// → '_x_x_x_'
'xxx'.replaceAll('', '_');
// → '_x_x_x_'
```

## TC39 meeting notes

- [November 2017](https://tc39.es/tc39-notes/2017-11_nov-28.html#10ih-stringprototypereplaceall-for-stage-1)
- [March 2019](https://github.com/tc39/tc39-notes/blob/master/meetings/2019-03/mar-26.md#stringprototypereplaceall-for-stage-2)
- [July 2019](https://github.com/tc39/tc39-notes/blob/master/meetings/2019-07/july-24.md#stringprototypereplaceall)
- [June 2020](https://github.com/tc39/notes/blob/master/meetings/2020-06/june-2.md#stringprototypereplaceall-for-stage-4)

## Specification

- [Ecmarkup source](https://github.com/tc39/proposal-string-replaceall/blob/master/spec.html)
- [HTML version](https://tc39.es/proposal-string-replaceall/)

## Implementations

- [SpiderMonkey](https://bugzilla.mozilla.org/show_bug.cgi?id=1540021), shipping in [Firefox 77](https://bugzilla.mozilla.org/show_bug.cgi?id=1608168)
- [JavaScriptCore](https://bugs.webkit.org/show_bug.cgi?id=202471), shipping in [Safari 13.1](https://webkit.org/blog/10247/new-webkit-features-in-safari-13-1/#javascript-improvements)
- [V8](https://bugs.chromium.org/p/v8/issues/detail?id=9801), shipping in Chrome 85
- Polyfills:
    - [core-js](https://github.com/zloirock/core-js#stringreplaceall)
    - [es-shims](https://github.com/es-shims/String.prototype.replaceAll)

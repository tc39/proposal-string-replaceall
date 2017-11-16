# String.prototype.replaceAll Proposal

## Status

Champion: Mathias Bynens (Google, @mathiasbynens)
This proposal is in stage 0 of [the TC39 process](https://tc39.github.io/process-document/).


## Motivation

Currently there is no way to replace all instances of a substring in a string without use of a global regexp.
`String.prototype.replace` only affects the first occurrence when used with a string argument. There is a lot of evidence that developers are trying to do this in JS — see the [StackOverflow question](https://stackoverflow.com/questions/1144783/how-to-replace-all-occurrences-of-a-string-in-javascript) with thousands of votes.

Currently the most common way of achieving this is to use a global regexp.

```js
var queryString = 'q=query+string+parameters';
var withSpaces = queryString.replace(/\+/g, ' ');
```

This has the disadvantage that special regexp characters must be escaped.

```js
var windowsStyle = 'windows\\style\\path';
const replacement = '\\';

// Only replaces the first instance.
var unixStyle = windowsStyle.replace(replacement, '/');
// --> "windows/style\path"

// Syntax Error
unixStyle = windowsStyle.replace(new RegExp(replacement, 'g'), '/');
```

## Proposed solution

We propose the addition of a new method to the String prototype - `replaceAll`. This would give developers a straight-forward way to accomplish this common, basic operation.

```js
var queryString = 'q=query+string+parameters';
var withSpaces = queryString.replaceAll('+', ' ');
```

It also removes the need to escape special regexp characters.

```js
var windowsStyle = 'windows\\style\\path';
const replacement = '\\';

var unixStyle = windowsStyle.replaceAll(replacement, '/');
// --> "windows/style/path"
```

## High-level API

`String.prototype.replaceAll(searchValue, replaceValue)`

Proposed semantics:

1. `searchValue` throws if it is a RegExp (there's no reason to use `replaceAll` with a RegExp `searchValue`). Otherwise, the remaining algorithm uses `ToString(searchValue)`.

Alternative 1: Unconditionally use `ToString(searchValue)`, even if `searchValue` is a RegExp. Doesn't seem like a good option since this will break RegExp args in unexpected ways (e.e. `/./.toString()  // "/[.]/"`).

Alternative 2: If `searchValue` is a RegExp, create a clone including the 'g' flag and dispatch to `RegExp.prototype[@@replace]`. Otherwise, use `ToString(searchValue)`.

## FAQ

## Illustrative examples


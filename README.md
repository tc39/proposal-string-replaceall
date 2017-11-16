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

## Proposed solution

We propose the addition of a new method to the String prototype - `replaceAll`. This would give developers a straight-forward way to accomplish this common, basic operation.

```js
var queryString = 'q=query+string+parameters';
var withSpaces = queryString.replaceAll('+', ' ');
```

## High-level API

## FAQ

## Illustrative examples


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

The proposed signature is the same as the existing `String.prototype.replace` method:

`String.prototype.replaceAll(searchValue, replaceValue)`

### `searchValue`

1. `searchValue` throws if it is a RegExp (there's no reason to use `replaceAll` with a RegExp `searchValue`). Otherwise, the remaining algorithm uses `ToString(searchValue)`. This option can be implemented very efficiently.

Alternative 1.1: Unconditionally use `ToString(searchValue)`, even if `searchValue` is a RegExp. Doesn't seem like a good option since this will break RegExp args in unexpected ways (e.e. `/./.toString()  // --> "/[.]/"`).

Alternative 1.2: If `searchValue` is a RegExp, create a clone including the 'g' flag and dispatch to `RegExp.prototype[@@replace]`. Otherwise, use `ToString(searchValue)`. There's precedent for this in `RegExp.prototype[@@split]`. This option seems consistent with user expectations; but we lose efficiency & simplicity on the implementation side, and we create an unexpected performance trap since cloning the regexp instance is slow.

### `replaceValue`

2. The algorithm uses `ToString(replaceValue)` and neither implements `GetSubstitution` semantics nor allows callable `replaceValue`. The `replace` function's interface is (perhaps unnecessarily) complex. We can take this opportunity to simplify, resulting in less cognitive load for users & simpler, more efficient implementations for VMs.

Alternative 2.1: As above, but implement `GetSubstitution` for more consistency with `replace`.
Alternative 2.2: As 2.1, but additionally allow callable `replaceValue` for more consistency with `replace`. Both 2.1 and 2.2 add complexity and overhead to the implementation.

### `RegExp.prototype[@@replaceAll]`

3. There will be no new `RegExp.prototype[@@replaceAll]` function. Depending on the chosen solution for `searchValue`, RegExp arguments either throw or are forwarded to `@@replace`.

## Comparison to other languages

TODO

## FAQ

Q: What are the main benefits? 

A: A simplified API for this common use-case that does not require RegExp knowledge. A way to global-replace strings without having to escape RegExp syntax characters. Possibly improved optimization potential on the VM side.

## Illustrative examples

TODO

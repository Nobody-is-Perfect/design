# Design of Nobody is Perfect

This repo is mostly for keeping track of design choices, like the API, ...

## Overall design

This will be a simple client-server architecture using websockets.

The frontend will be written in JS using React, the backend will be written in Kotlin.

## Questions

Mostly still an open question. @Sam will look into this.

### Format

Example:

```
{Bern} is the capital of Switzerland.
```

The answer is provided encapsulated with `{}`, this will replaced with a `_` in the UI.

This is heavily influenced by the cloze-cards of Anki.

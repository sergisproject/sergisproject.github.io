---
layout: sergis
title: Author Developer Documentation
legacy: 1

extrastyle: 'td > em { white-space: nowrap; }'

sidebartitle: Table of Contents
sidebar:
- name: Backends
  href: "#backends"
---
# Author Developer Documentation

## Backends

In the SerGIS Prompt Author, a backend is a JavaScript library that handles user login (if applicable) and game (file) storage.

Each function in the backend must return a [JavaScript Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Each backend is a JavaScript object. It must be assigned to a JavaScript variable `AUTHOR_BACKEND`. This object must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `getGameList` | none | Promise&lt;Object&lt;string, Date&gt;&gt; | Get a list of all the user's games, returned in an object where the keys are the game names and the values are the "Last Modified" dates for each game.
| `loadGame` | *`gameName` (string)* | Promise&lt;Object&gt; | Get the JSON data for a specific game, identified by its name.
| `saveGame` | *`gameName` (string),* *`jsondata` (Object)* | Promise | Save new JSON data for a specific game, identified by name. (If the name already exists, it is overwritten.)
| `removeGame` | *`gameName` (string)* | Promise | Remove a specific game, identified by its name.

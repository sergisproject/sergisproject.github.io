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
| `init` | none | Promise | Initialize the backend (only called once, after the page is loaded).
| `getGameList` | none | Promise&lt;Object&lt;string, Date&gt;&gt; | Get a list of all the user's games, returned in an object where the keys are the game names and the values are the "Last Modified" dates for each game.
| `loadGame` | *`gameName` (string)* | Promise&lt;Object&gt; | Get the JSON data for a specific game, identified by its name.
| `saveGame` | *`gameName` (string),* *`jsondata` (Object)* | Promise | Save new JSON data for a specific game, identified by name. (If the name already exists, it is overwritten.)
| `renameGame` | *`gameName` (string),* *`newGameName` (string)* | Promise | Rename a game from an old name to a new name.
| `removeGame` | *`gameName` (string)* | Promise | Remove a specific game, identified by its name.
| `checkGameName` | *`gameName` (string)* | Promise&lt;number&gt; | Check whether a certain game name is valid for a new game. This should return `0` if the game name is already taken, `-1` if the game name is invalid, or `1` if the game name is all good and dandy.
| `previewGame` (optional) | *`gameName` (string)* | Promise&lt;Object&gt; | Open up a preview of a game, identified by its name.
| `publishGame` (optional) | *`gameName` (string)* | Promise&lt;Object&gt; | Publish a game, identified by its name.

The Promises returned by `previewGame` and `publishGame` are resolved with an object representing an HTTP request. This object has the following properties:

| Property Name | Type | Value
| ------------- | ---- | -----
| `url` | *string* | The URL to open.
| `method` | *string* | The HTTP method to use for the request (default: "GET").
| `data` | *object* | Any URL parameters or POST data to send with the request (default: {}).

---
layout: sergis
title: Author Developer Documentation

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

Each backend is a JavaScript object. It must be assigned to a JavaScript variable `AUTHOR.BACKEND`. This object must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `init` | *`onPromptLock` (function),* *`onGameUpdate` (function)* | Promise | Initialize the backend (only called once, after the page is loaded). The `onPromptLock` and `onGameUpdate` are functions that should be called when a prompt in the last-loaded game is locked by a different user (`onPromptLock`) or when a different user updates the game data (`onGameUpdate`). For more on what should be passed to these functions, see "Multiple Users" below.
| `getGameList` | none | Promise&lt;Object&lt;string, Object&gt;&gt; | Get a list of all the user's games, returned in an object where each key is the game name and the corresponding value is an AuthorGame object (see below).
| `renameGame` | *`gameName` (string),* *`newGameName` (string)* | Promise | Rename a game from an old name to a new name. Must reject the promise if any user has the game open.
| `shareGame` (optional) | *`gameName` (string),* *`username` (string)* | Promise | Share one of the user's games with a different user.
| `unshareGame` (optional) | *`gameName` (string),* *`username` (string)* | Promise | Remove a user from the list of users that a game is shared with.
| `removeGame` | *`gameName` (string)* | Promise | Remove a specific game, identified by its name. Must reject the promise if any user has the game open.
| `checkGameName` | *`gameName` (string)* | Promise&lt;number&gt; | Check whether a certain game name is valid for a new game. This should return `0` if the game name is already taken, `-1` if the game name is invalid, or `1` if the game name is all good and dandy.
| `loadGame` | *`gameName` (string)* | Promise&lt;Object&gt; | Get the JSON data for a specific game, identified by its name. If the game name refers to a game that does not yet exist, it must be created. The backend must store this game as the "current" game for when functions such as `saveCurrentGame` are called.
| `saveCurrentGame` | *`jsondata` (Object),* *`path` (string)* | Promise | Save new JSON data for the current game. If `path` is defined, then it is a dot-separated path to the part of the JSON that should be updated (for example, if `path` is `"promptList.5.prompt"`, then `jsondata` should be assigned to `promptList[5].prompt`.
| `previewCurrentGame` (optional) | none | Promise&lt;Object&gt; | Open up a preview of the current game.
| `publishCurrentGame` (optional) | none | Promise&lt;Object&gt; | Publish the current game.

An AuthorGame object is an object with these properties:

| Property Name | Type | Value
| ------------- | ---- | -----
| `lastModified` | *Date* | The date that the game was last modified.
| `owner` (optional) | *string* | The owner of the game. If the owner is the logged-in user, then this property should be null or not provided.
| `sharedWith` (optional) | *Object&lt;string, string&gt;* | Any other users with whom the game is shared. Each key in the object is a user's username, and each value is the user's display name. If the game is not owned by the logged-in user, then this property should be null or not provided.
| `allowSharing` (optional) | *boolean* | Whether to allow the user to share the game with others. If the game is not owned by the logged-in user, then this property should be falsy. (Default: `false`)

The Promises returned by `previewGame` and `publishGame` are resolved with an object representing an HTTP request (which is opened in an iframe or a new window). This object has the following properties:

| Property Name | Type | Value
| ------------- | ---- | -----
| `url` | *string* | The URL to open.
| `method` | *string* | The HTTP method to use for the request (default: "GET").
| `data` | *object* | Any URL parameters or POST data to send with the request (default: {}).
| `enctype` | *string* | The encoding type for any POST data (default: "" if the method is "GET", or "application/x-www-form-urlencoded" otherwise).

### Multiple Users

The backend may optionally support multiple users working on the same game.
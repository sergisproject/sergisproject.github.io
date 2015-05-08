---
layout: sergis
title: Author Developer Documentation

extrastyle: 'td > em { white-space: nowrap; }'
---
# Author Developer Documentation

## Backends

In the SerGIS Prompt Author, a backend is a JavaScript library that handles user login (if applicable) and game (file) storage.

Each function in the backend must return a [JavaScript Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Some of these functions use a "JSON path". This is a simple dot-separated notation for specifying a point within a game's JSON data (it is not exactly JavaScript's object-property notation). Example: For the 5th prompt index's map, the path would be: `promptList.5.prompt.map`

Each backend is a JavaScript object. It must be assigned to a JavaScript variable `AUTHOR.BACKEND`. This object must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `init` | *`onPromptLock` (function),* *`onGameUpdate` (function)* | Promise | Initialize the backend (only called once, after the page is loaded). The `onPromptLock` and `onGameUpdate` are functions that should be called when a prompt in the last-loaded game is locked by a different user (`onPromptLock`) or when a different user updates the game data (`onGameUpdate`). For more on what should be passed to these functions, see "Multiple Users" below.
| `getUserList` (optional) | none | Promise&lt;Array&lt;[AuthorUser][AuthorUser]&gt;&gt; | Get a list of all the users that a user can share with (the user will also be able to enter a username). See "Multiple Users" below.
| `getGameList` | none | Promise&lt;Array&lt;[AuthorGame][AuthorGame]&gt;&gt; | Get a list of all the user's games, returned in an an array of AuthorGame objects (see below).
| `renameGame` | *`gameName` (string),* *`newGameName` (string)* | Promise | Rename a game from an old name to a new name. Must reject the promise if any user has the game open.
| `shareGame` (optional) | *`gameName` (string),* *`username` (string)* | Promise | Share one of the user's games with a different user. See "Multiple Users" below.
| `unshareGame` (optional) | *`gameName` (string),* *`username` (string)* | Promise | Remove a user from the list of users that a game is shared with. See "Multiple Users" below.
| `removeGame` | *`gameName` (string)* | Promise | Remove a specific game, identified by its name. Must reject the promise if any user has the game open.
| `checkGameName` | *`gameName` (string)* | Promise&lt;number&gt; | Check whether a certain game name is valid for a new game. This should return `0` if the game name is already taken, `-1` if the game name is invalid, or `1` if the game name is all good and dandy.
| `loadGame` | *`gameName` (string),* *`ownerUsername` (string)* | Promise&lt;Object&gt; | Get the JSON data for a specific game, identified by its name (and, if it's not our game, its owner's username). The return object has 2 properties: `jsondata` and `lockedPrompts` (see Multiple Users below; `lockedPrompts` takes the same object as `onPromptLock`). If the game name refers to a game that does not yet exist, it must be created. The backend must store this game as the "current" game for when functions such as `saveCurrentGame` are called.
| `saveCurrentGame` | *`jsondata` (Object),* *`path` (string)* | Promise | Save new JSON data for the current game. If `path` is defined, then it is a dot-separated JSON path (see top of page) that should be updated (for example, if `path` is `"promptList.5.prompt"`, then `jsondata` should be assigned to `promptList[5].prompt`.
| `previewCurrentGame` (optional) | none | Promise&lt;[AuthorRequest](#sergis-author-request-object)&gt; | Open up a preview of the current game.
| `publishCurrentGame` (optional) | none | Promise&lt;[AuthorRequest](#sergis-author-request-object)&gt; | Publish the current game.
| `lockCurrentPrompt` (optional) | *`promptIndex` (number)* | Promise | Lock a certain prompt for this user to edit. If another user has the prompt locked (i.e. is editing it), this promise must be rejected. See "Multiple Users" below.
| `unlockCurrentPrompt` (optional) | *`promptIndex` (number)* | Promise | Unlock a certain prompt that this user currently has locked, allowing other users to edit it. See "Multiple Users" below.

### SerGIS Author Game Object (`AuthorGame`)

An AuthorGame object is an object with these properties:

| Property Name | Type | Value
| ------------- | ---- | -----
| `name` | *string* | The name of the game.
| `lastModified` | *Date* | The date that the game was last modified.
| `owner` (optional) | *[AuthorUser][AuthorUser]* | The owner of the game. If the owner is the logged-in user, then this property must be null or not provided.
| `allowSharing` (optional) | *boolean* | Whether to allow the user to share the game with others. If the game is not owned by the logged-in user, then this property should be falsy. (Default: `false`)
| `sharedWith` (optional) | *Array&lt;[AuthorUser][AuthorUser]&gt;* | Any other users with whom the game is shared. If the game is not owned by the logged-in user, then this property should be null or not provided.
| `currentlyEditing` (optional) | *Array&lt;[AuthorUser][AuthorUser]&gt;* | Any **other** users that are currently editing this game (not including the instance of any game that this user in this session is editing).

### SerGIS Author Request Object (`AuthorRequest`)

An AuthorRequest object is an object that represents an HTTP request (which is opened in an iframe or a new window). It has the following properties:

| Property Name | Type | Value
| ------------- | ---- | -----
| `url` | *string* | The URL to open.
| `method` | *string* | The HTTP method to use for the request (default: "GET").
| `data` | *object* | Any URL parameters or POST data to send with the request (default: {}).
| `enctype` | *string* | The encoding type for any POST data (default: "" if the method is "GET", or "application/x-www-form-urlencoded" otherwise).

This is used as the return value for the `previewCurrentGame` and `publishCurrentGame` functions.

### SerGIS Author User Object (`AuthorUser`)

A SerGIS Author User Object is an object with the following properties:

| Property Name | Type | Value
| ------------- | ---- | -----
| `username` | *string* | The username of the user.
| `displayName` (optional) | *string* | The display name of the user.
| `groupName` (optional) | *string* | The name of a group that this user is a part of. (Used to organize the user dropdown in the "Share Game" dialog.)

### Multiple Users

The backend may optionally support multiple users working on the same game, or multiple instances of the same user working on the same game (this is still counted as "multiple users").

To support multiple users, the backend should implement these things:

1. **Locking:** Allows users to "lock" the prompt that they're currently editing.
   Requires:
   - `lockCurrentPrompt` function
   - `unlockCurrentPrompt` function
   - `init` function's `onPromptLock` and `onGameUpdate` parameters (see below)
   - `loadGame` function's `lockedPrompts` property in the return object
1. **Sharing:** Allows users to share games with other users.
   Requires:
   - Locking
   - `getUserList` function (not necessary, but should be provided)
   - `shareGame` function
   - `unshareGame` function
   - `loadGame` function's 2nd parameter (`ownerUsername`)
   - AuthorGame's `owner`, `sharedWith`, and `allowSharing` properties

**`init` function's parameters**

`onPromptLock`: This is a function that must be called with an object representing which prompts are locked by which users. It must be passed an object whose keys are locked prompt indexes and whose values are a string representing the display name of the user who has the prompt locked.

`onGameUpdate`: This is a function that must be called whenever another user updates the current game's JSON data. It is called with 2 parameters: `jsondata` (the new JSON game data, or a section of it), and `path` (an optional JSON path to where the `jsondata` should go; see JSON paths at the top of this page).



[AuthorGame]: #sergis-author-game-object-authorgame
[AuthorRequest]: #sergis-author-request-object-authorrequest
[AuthorUser]: #sergis-author-user-object-authoruser

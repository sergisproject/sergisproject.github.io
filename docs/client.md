---
layout: sergis
title: Client Documentation

extrastyle: 'td > em { white-space: nowrap; }'

sidebartitle: Table of Contents
sidebar:
- name: Promises in Frontends and Backends
  href: "#promises-in-frontends-and-backends"
- name: Frontends
  href: "#frontends"
- name: Backends
  href: "#backends"
---
# Client Documentation

<p style="text-align: center;"><a href="https://docs.google.com/drawings/d/1aDEHLen7vmv6BJ2mfVee2BMB6y3Zp8NhDK-hQRfoTIc/edit?usp=sharing" target="_blank"><img src="server-client.png" style="border: 1px solid black; padding: 10px; border-radius: 5px;"></a></p>

## Promises in Frontends and Backends

Any property of a frontend or backend that is a function **must return a *JavaScript Promise* that is resolved with the value in question** (or resolved with no data if the function does not return a value) after the function has completed.

**In the case of an error, the function should reject the promise with a human-readable error message (starting with a capital letter and ending with a period).**

Calling a function that returns a promise should look something like this:

    getSomething().then(function (value) {
        // ...
    }).catch(sergis.error);

If you want to do something special if the promise is rejected (as opposed to just reporting/alerting it through sergis.error), do something like this:

    getSomething().then(function (value) {
        // ...
    }, function (error) {
        // ...
    }).catch(sergis.error);

**Never forget the `.catch(sergis.error)` on the end, just in case one of the `then` functions throws an error!**

## Frontends

In SerGIS, a frontend is a JavaScript library for the SerGIS Web Client that handles the rendering of the map and the rendering of Map [Action objects][actionobject] on the map. This is separate to easily allow different mapping APIs and libraries to be used.

Each frontend is a JavaScript object. It must be assigned to `sergis.frontend`.

This object must have the following properties:

| Property | Type   | Value
| -------- | ------ | -----
| `name`   | string | The name of the frontend (usually matches the frontend filename without `.js` on the end).

This object must also have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `init` | *`mapContainer` (DOM element),* *`map` ([Map][mapobject])* | Promise&lt;array&gt; | Initialize the map within the DOM element mapContainer, centering it based on the [SerGIS Map object][mapobject] provided. If successful, the Promise should be resolved to an array representing toolbar buttons to show in the toolbox at the top. Each button is represented by a n object with a `label` property (which is a [SerGIS Content object][contentobject]) and an optional `tooltip` property (a string).
| `reinit` (optional) | *`mapContainer` (DOM element),* *`map` ([Map][mapobject])* | Promise | Re-initialize the map within the DOM element mapContainer, centering it based on the [SerGIS Map object][mapobject] provided. If not provided, then `init` is called if the map has to be re-initialized.
| `centerMap` | *`map` ([Map][mapobject]),* | Promise | Center the map on the given coordinates (provided as numbers) and zoom to the given zoom value (an integer).
| `toolbarAction` | *`buttonIndex` (number)* | Promise | Execute the toolbar action for the toolbar button at buttonIndex. (Only ever called if `init` included at least one item in the array that its promise was resolved to. The buttonIndex refers to the index of the toolbar button in this array.)

This object must also have a property named `actions`, which is an object with the actions that can be taken on the map. It must have a function for each Map Action that could be present in an Map [Action object][actionobject] (not including Gameplay Actions such as `explain`, `goto`, and `logout`). The function name corresponds to the action's `name`, and the function's parameters correspond to data passed in the action's `data` array.

## Backends

In SerGIS, a backend is a JavaScript library for the SerGIS Web Client that handles the interaction between the server and the client UI (or the interaction between a [SerGIS JSON Game Data file][sergis-json-game-data] and the client UI).

If a server is used, the server that is used through the backend must keep track of the actions that the user has completed so far (and which prompt index the user is on). This is separate from the main SerGIS code to allow the implementation of different backends corresponding to different server-side programs.

Each backend is a JavaScript object. It must be assigned to `sergis.backend`. This object must have 2 properties, `account` and `game`, which hold the functions for the backend.

- `account` must have the following functions:

  | Function Name | Arguments | Return Value | Description
  | ------------- | --------- | ------------ | -----------
  | `logIn` | *`username` (string),* *`password` (string)* | Promise&lt;User&gt; | Attempt to log in with the provided username and password. If successful, resolve the promise with a User object (below); if unsuccessful, reject the promise with an error message.
  | `logOut` | none | Promise | Log the user out.
  | `getUser` | none | Promise&lt;User&gt; | Get the logged-in user (reject the promise if nobody is logged in).

  > In these functions, `User` is an object representing a SerGIS user. It includes personal attributes and options regarding game-play. It has the following properties:
  >
  > | Property      | Type   | Value
  > | ------------- | ------ | -----
  > | `displayName` | string | The display name of the user.
  > | `promptIndex` (optional) | number | Which prompt the user is on (in the case of a previously started session). If not provided, it is assumed to be the first prompt (`0`).
  > | `jumpingBackAllowed` (optional) | boolean | Whether the user is allowed to go back to previously answered prompts. See `jumpingBackAllowed` in the [SerGIS JSON Game Data][sergis-json-game-data] reference.
  > | `jumpingForwardAllowed` (optional) | boolean | Whether the user is allowed to skip prompts and come back to them later. See `jumpingForwardAllowed` in the [SerGIS JSON Game Data][sergis-json-game-data] reference.

- `game` must have the following functions (NOTE: none of these will be called until a user is logged in):

  | Function Name | Arguments | Return Value | Description
  | ------------- | --------- | ------------ | -----------
  | `getPreviousMapActions` | none | Promise&lt;array&lt;[Action][actionobject]&gt;&gt; | Get a list of all the previous Map Actions (in order, with the most recent last) that the user has chosen up to this point. (This is used if the SerGIS UI has to re-draw the actions on the map, e.g. if the user is restarting the session or if the user is going back to a previous prompt.) This array must NEVER include Gameplay Actions (i.e. `explain`, `goto`, or `logout`). If the backend uses [SerGIS JSON Game Data][sergis-json-game-data], then this should respect `showActionsInUserOrder`. Also, this should NEVER return actions previously chosen by the user for the current prompt.
  | `getPromptCount` | none | Promise&lt;number&gt; | Get the total number of prompts.
  | `getPrompt` | *`promptIndex` (number)* | Promise&lt;[Prompt][promptobject]&gt; | Go to a prompt index and returns the Prompt object representing the question or information. (Make sure to check on the server if the user has permission to go to this prompt; even if `jumpingBackAllowed` and/or `jumpingForwardAllowed` aren't true, anything on the client side of things can be manipulated.) Also, this function should save the current state on the server (i.e. which prompt the user is on) so the user can resume where he or she left off, and, if the backend uses [SerGIS JSON Game Data][sergis-json-game-data], this should respect `onBackwardJump`.
  | `getActions` | *`promptIndex` (number),* *`choiceIndex` (number)* | Promise&lt;array&lt;[Action][actionobject]&gt;&gt; | Get the action(s) for a specific choice (choiceIndex) of a prompt (promptIndex). The server should store the user's response so it can be retrieved later using `getPreviousMapActions()`.


[actionobject]:  json.html#action-object  "SerGIS JSON Action Object"
[contentobject]: json.html#content-object "SerGIS JSON Content Object"
[promptobject]:  json.html#prompt-object  "SerGIS JSON Prompt Object"
[mapobject]:     json.html#map-object     "SerGIS JSON Map Object"

[sergis-json-game-data]: json.html#sergis-json-game-data "SerGIS JSON Game Data"

[frontends]: client.html#frontends "SerGIS Client Frontends"
[backends]:  client.html#backends  "SerGIS Client Backends"

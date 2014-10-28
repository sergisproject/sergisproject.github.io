---
layout: sergis
title: Documentation

extrastyle: 'td > em { white-space: nowrap; }'

sidebartitle: Table of Contents
sidebar:
- name: SerGIS Object Reference
  href: "#sergis-object-reference"
  subitems:
  - name: Action Object
    href: "#action-object"
  - name: Content Object
    href: "#content-object"
  - name: Prompt Object
    href: "#prompt-object"
- name: Promises in Frontends and Backends
  href: "#promises-in-frontends-and-backends"
- name: Frontends
  href: "#frontends"
- name: Backends
  href: "#backends"
---
# Documentation

<p style="text-align: center;"><img src="server-client.png" style="border: 1px solid black; padding: 10px; border-radius: 5px;"></p>

## SerGIS Object Reference

### Action Object

An object representing an action to do on the map.

| Property | Type   | Value
| -------- | -----  | -----
| `name`   | string | The name of the action to perform.
| `data`   | array  | Any data to pass as parameters to the action function.

The name of the action (`name`) must correspond to a function in the `sergis.frontend.actions` object (see [Frontends](#frontends) below). Additionally, any of these special values may be used:

 - `goto`: Go to a specific prompt (`data` should have 1 item: the prompt index to go to).
 - `continue`: Move on to the next prompt without performing an action (`data` not required).
 - `logout`: Log the user out (`data` not required).

If a normal frontend action is specified, it will automatically advance to the next prompt. The 3 special actions listed above cannot be combined with any other actions (i.e. if one of them is returned through `sergis.backend.getActions()`, it should be the only action returned). None of the special actions should be returned via `sergis.backend.getPreviousActions()`.

### Content Object

An object representing some sort of content that is part of a prompt or choice. It must have a `type` property that identifies what type of content it is. The value of the `type` property determines which other properties are present. The main value for the type is stored in `value` (which is required). Some types have other possible properties too, all of which are optional (their default values are shown).

 - **Text Type:** `{type: "text", value: "plain text here", centered: false}`
 - **HTML Type:** `{type: "html", value: "<p>HTML content here</p>"}`
 - **Image Type:** `{type: "image", value: "URL of image"}`
 - **YouTube Type:** `{type: "youtube", value: "youtube-video-id-here", width: 400, height: 300, playerVars: {autohide: 1}}`

### Prompt Object

An object representing either a question for the user or information to show the user.

| Property  | Type   | Value
| --------  | ----   | -----
| `title`   | string | A text-only title for the prompt (usually just the general topic of the question or information).
| `map`     | object | An object with 3 properties: `latitude` (number), `longitude` (number), `zoom` (number). If any are not provided, or if `map` is not provided, then the previous values are used. **MUST be provided for the first prompt**, and **must be provided for all prompts if jumping is allowed** (see [Backends](#backends) below).
| `content` | array&lt;[Content][contentobject]&gt; | The content of the prompt. Each array item must be a Content object.
| `choices` | array&lt;[Content][contentobject]&gt; | A list of the possible choices for the prompt. Each item must be a Content object that represents the choice. (NOTE: Unlike the `content` property, only one Content object can be provided for each choice.) If not provided, or if the array is empty, a "Continue" button is shown if it is not the last prompt. (This may be useful if the prompt just provides information instead of asking a question.)

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

In SerGIS, a frontend is a JavaScript library for the SerGIS client that handles the rendering of the map and the rendering of Action objects on the map. This is separate to easily allow different mapping APIs and libraries to be used.

Each frontend is a JavaScript object. It must be assigned to `sergis.frontend`. This object must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `init` | *`mapContainer` (DOM element),* *`latitude` (number),* *`longitude` (number),* *`zoom` (number)* | Promise | Initialize the map within the DOM element mapContainer, centering it on the given coordinated (provided as numbers) and zoomed to the given zoom value (an integer).
| `centerMap` | *`latitude` (number),* *`longitude` (number),* *`zoom` (number)* | Promise | Center the map on the given coordinates (provided as numbers) and zoom to the given zoom value (an integer).

It must also have a property named `actions`, which is an object with the actions that can be taken on the map. It must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `buffer` | *...*, *...* | Promise | ...
| `...` | *...*, *...* | Promise | ...

## Backends

In SerGIS, a backend is a JavaScript library for the SerGIS client that handles the interaction between the server and the client UI. The server that is used through the backend must keep track of the actions that the user has completed so far (and which prompt the user is on). This is separate from the main SerGIS code to allow the implementation of different backends corresponding to different server-side programs.

Each backend is a JavaScript object. It must be assigned to `sergis.backend`. This object must have 2 properties, `account` and `game`, which hold the functions for the backend.

`account` must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `logIn` | *`username` (string),* *`password` (string)* | Promise&lt;string&gt; | Attempt to log in with the provided username and password. If successful, it should resolve the promise with the display name.
| `logOut` | none | Promise | Log the user out.
| `getUser` | none | Promise&lt;string&gt; | Get the user's display name.

`game` must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `isJumpingAllowed` | none | Promise&lt;boolean&gt; | If resolved to `true`, allows the user to skip around between different prompts. (If `false`, the user will only be allowed to proceed through prompts in a forward, sequential order, without going back after making a choice.)
| `getPreviousActions` | none | Promise&lt;array&lt;[Action][actionobject]&gt;&gt; | Get a list of all the previous actions (in order, with the most recent last) that the user has chosen up to this point (used if the SerGIS UI has to re-draw the actions on the map, e.g. if the user is restarting the session or if the user is going back to a previous prompt).
| `getPromptCount` | none | Promise&lt;number&gt; | Get the total number of prompts.
| `getPrompt` | *`promptIndex` (number)* | Promise&lt;[Prompt][promptobject]&gt; | Go to a prompt number and returns the Prompt object representing the question or information. (Make sure to check on the server if the user has permission to go to this prompt; even if `allowJumpingAround` is false, anything on the client side of things can be manipulated.) Also, this function should save the current state on the server (i.e. which prompt the user is on) so the user can resume where he or she left off. **NOTE: The prompt number (promptIndex) starts at 1, not 0!**
| `getActions` | *`promptIndex` (number),* *`choiceIndex` (number)* | Promise&lt;array&lt;[Action][actionobject]&gt;&gt; | Get the actions for a specific prompt number and choice index within that prompt. (The server should store the user's response so it can be retrieved later using `getPreviousActions()`.)


[actionobject]: #action-object "SerGIS Action Object"
[contentobject]: #content-object "SerGIS Content Object"
[promptobject]: #prompt-object "SerGIS Prompt Object"

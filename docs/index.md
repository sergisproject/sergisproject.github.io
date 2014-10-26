---
layout: sergis
title: Documentation
---
# Documentation

<p style="text-align: center;"><img src="server-client.png" style="border: 1px solid black; padding: 10px; border-radius: 5px;"></p>

## SerGIS Object Reference

### Action Object

An object representing an action to do on the map.

| Property | Type   | Value
| -------- | -----  | -----
| `name`   | string |  The name of the action to perform (corresponding to a function in the `sergis.frontend.actions` object; see [Frontends](#frontends) below).
| `data`   | array  | Any data to pass as parameters to the action function.

### Content Object

An object representing some sort of content that is part of a question or answer. It must have a `type` property that identifies what type of content it is. The value of the `type` property determines which other properties are present.

 - **Text Type:** `{type: "text", value: "plain text here"}`
 - **HTML Type:** `{type: "html", value: "<p>HTML content here</p>"}`
 - **Image Type:** `{type: "image", value: "URL of image"}`
 - **YouTube Type:** `{type: "youtube", value: "youtube-video-id-here"}`

### Question Object

An object representing a question for the user.

| Property  | Type   | Value
| --------  | ----   | -----
| `title`   | string | A text-only title for the question.
| `content` | array&lt;[Content](#content-object "SerGIS Content Object")&gt; | The content of the question. Each array item must be a Content object.
| `answers` | array&lt;[Content](#content-object "SerGIS Content Object")&gt; | A list of the possible answers for the question. Each item must be a Content object that represents the answer. (NOTE: Unlike the `content` property, only one Content object can be provided for each answer.)

## JS Promises in Frontends and Backends

Any property of a frontend or backend that is a function **must return a *JavaScript Promise* that is resolved with the value in question** (or resolved with no data if the function does not return a value) after the function has completed.

**In the case of an error, the function should reject the promise with a human-readable error message (starting with a capital letter and ending with a period).**

## Frontends

In SerGIS, a frontend is a JavaScript library for the SerGIS client that handles the rendering of the map and the rendering of Action objects on the map. This is separate to easily allow different mapping APIs and libraries to be used.

Each frontend is a JavaScript object. It must be assigned to `sergis.frontend`. This object must have the following functions:

| Function Name | Arguments | Return Value | Description
| -------- | ---- | ----- | -----
| `init` | `mapContainer` (DOM element), <br> `latitude` (number), <br> `longitude` (number), <br> `zoom` (number) | Promise | Initialize the map within the DOM element mapContainer, centering it on the given coordinated (provided as numbers) and zoomed to the given zoom value (an integer).
| `centerMap` | `latitude` (number), <br> `longitude` (number), <br> `zoom` (number) | Promise | Center the map on the given coordinates (provided as numbers) and zoom to the given zoom value (an integer).

It must also have a property named `actions`, which is an object with the actions that can be taken on the map. It must have the following functions:

| Function Name | Arguments | Return Value | Description
| ---- | ---- | ---- | ----
| `buffer` | ... | Promise | ...
| `...` | ... | Promise | ...

## Backends

In SerGIS, a backend is a JavaScript library for the SerGIS client that handles the interaction between the server and the client UI. The server that is used through the backend must keep track of the actions that the user has completed so far (and which question the user is on). This is separate from the main SerGIS code to allow the implementation of different backends corresponding to different server-side programs.

Each backend is a JavaScript object. It must be assigned to `sergis.backend`. This object must have 2 properties, `account` and `game`, which hold the functions for the backend.

`account` must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `logIn` | `username` (string), <br> `password` (string) | Promise&lt;string&gt; | Attempt to log in with the provided username and password. If successful, it should resolve the promise with the display name.
| `logOut` | none | Promise | Log the user out.
| `getUser` | none | Promise&lt;string&gt; | Get the user's display name.

`game` must have the following functions:

| Function Name | Arguments | Return Value | Description
| ------------- | --------- | ------------ | -----------
| `isJumpingAllowed` | none | Promise&lt;boolean&gt; | If resolved to `true`, allows the user to skip around different questions. (If false, the user will only be allowed to proceed through questions in a forward, sequential order, without going back after making a choice.)
| `getPreviousActions` | none | Promise&lt;array&lt;Action&gt;&gt; | Get a list of all the previous actions (in order, with the most recent last) that the user has chosen up to this point (used if the SerGIS UI has to re-draw the actions on the map, e.g. if the user is restarting the session or if the user is going back to a previous question).
| `getQuestionCount` | none | Promise&lt;number&gt; | Get the total number of questions.
| `getQuestion` | `questionIndex` (number) | Promise&lt;Question&gt; | Go to a question number and returns the Question object representing the question. (Make sure to check on the server if the user has permission to go to this question; even if `allowJumpingAround` is false, anything on the client side of things can be manipulated.) Also, this function should save the current state on the server (i.e. which question the user is on) so the user can resume where he or she left off.
| `getAction` | `questionIndex` (number), <br> `answerIndex` (number) | Promise&lt;Action&gt; | Get the action for a specific question number and answer number within that question. (The server should store the user's response so it can be retrieved later using `getPreviousActions()`.)

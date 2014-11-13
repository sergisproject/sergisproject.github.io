---
layout: sergis
title: JSON Format Documentation

extrastyle: 'td > em { white-space: nowrap; }'

sidebartitle: Table of Contents
sidebar:
- name: SerGIS JSON Object Reference
  href: "#sergis-json-object-reference"
  subitems:
  - name: Action Object
    href: "#action-object"
  - name: Content Object
    href: "#content-object"
  - name: Prompt Object
    href: "#prompt-object"
- name: SerGIS JSON Game Data
  href: "#sergis-json-game-data"
---
# SerGIS JSON Format Documentation

<p style="text-align: center;"><a href="https://docs.google.com/drawings/d/1aDEHLen7vmv6BJ2mfVee2BMB6y3Zp8NhDK-hQRfoTIc/edit?usp=sharing" target="_blank"><img src="server-client.png" style="border: 1px solid black; padding: 10px; border-radius: 5px;"></a></p>

SerGIS has a special JSON format that is used to store its data. This JSON content is not usually directly available on the client-side, but rather it is used on the server-side so the server can push the data to the client as needed (or directly used by the backend if no server-side system is used).

## SerGIS JSON Object Reference

These objects are referenced below in the JSON spec.

### Action Object

A SerGIS JSON Action Object is an object representing an action to do on the map.

| Property | Type   | Value
| -------- | -----  | -----
| `name`   | string | The name of the action to perform.
| `data`   | array  | Any data to pass as parameters to the action function. If the action function does not take any parameters, you can leave this out.

The `name` of the action can be any of these actions:

 - `buffer`
 - ...
 - ...
 - ...

There are also 3 "special" actions that do not affect the map, but rather affect the game sequence. They cannot be combined with the "normal" actions listed above. The "special" actions are:

 - `goto`: Go to a specific prompt (`data` should have 1 item: the prompt index to go to).
 - `continue`: Move on to the next prompt without performing an action (`data` not required).
 - `logout`: Log the user out (`data` not required).

### Content Object

A SerGIS JSON Content Object is an object representing some sort of content that is part of a prompt or choice.

It must have a `type` property that identifies what type of content it is. The value of the `type` property determines which other properties are present.

The main value for the type is stored in `value` (which is required). Some types have other possible properties too, all of which are optional (their default values are shown).

 - **Text Type:** `{type: "text", value: "plain text here", centered: false}`
 - **HTML Type:** `{type: "html", value: "<p>HTML content here</p>"}`
 - **Image Type:** `{type: "image", value: "URL of image"}`
 - **YouTube Type:** `{type: "youtube", value: "youtube-video-id-here", width: 400, height: 300, playerVars: {autohide: 1}}`

### Prompt Object

A SerGIS JSON Prompt Object is an object representing either a question for the user or information to show the user.

| Property  | Type   | Value
| --------  | ----   | -----
| `title`   | string | A text-only title for the prompt (usually just the general topic of the question or information).
| `map`     | object | An object with 3 properties: `latitude` (number), `longitude` (number), `zoom` (number). If any are not provided, or if `map` is not provided, then the previous values are used. **MUST be provided for the first prompt**, and **should be provided for all prompts if jumping is allowed** (see [Backends](#backends) below).
| `contents` | array&lt;[Content][contentobject]&gt; | The content of the prompt. Each array item must be a Content object.
| `choices` (optional) | array&lt;[Content][contentobject]&gt; | A list of the possible choices for the prompt. Each item must be a Content object that represents the choice. (NOTE: Unlike in the `contents` property, only one Content object can be provided for each choice.) If not provided, or if empty, a "Continue" button is shown if it is not the last prompt. (This may be useful if the prompt just provides information instead of asking a question.)

## SerGIS JSON Game Data

SerGIS JSON Game Data is a JSON file with a specific structure. The JSON data consists of an object with the following properties:

| Property | Type | Value
| -------- | ---- | -----
| `jumpingAllowed` | boolean | Whether the user should be allowed to jump between questions at will. If false, the user can only answer questions in a forward, continuous fashion.
| `promptList` | array | An array of objects representing the different prompts and choices.

Each object in the `promptList` array has the following properties:

| Property | Type | Value
| -------- | ---- | -----
| `prompt` | [Prompt][promptobject] | The SerGIS Prompt object representing the prompt.
| `actionList` (optional) | array | An array of objects representing different actions. Each item in this array corresponds to a choice in the `prompt.choices` array. If `prompt.choices` is empty or not provided, then this can be empty or not provided. (This is separate from `prompt` so a server can send `prompt` on to the client without revealing which choice is best.)

Each object in the `actionList` array has the following properties:

| Property | Type | Value
| -------- | ---- | -----
| `actions` | array<[Action][actionobject]> | An array of SerGIS Action objects representing the actions to be taken if this choice is selected. (Actions are evaluated in the order that they appear in this array.) After these actions are taken, the game will advance to the next prompt automatically (unless otherwise instructed).
| `pointValue` (optional) | number | The amount of points that the user should have added to his score for choosing this choice. If not provided, defaults to `0`.
| `explanation` (optional) | [Content][contentobject] | A SerGIS Content object that offers an explanation as to why this choice was correct or incorrect.



[actionobject]: #action-object "SerGIS JSON Action Object"
[contentobject]: #content-object "SerGIS JSON Content Object"
[promptobject]: #prompt-object "SerGIS JSON Prompt Object"

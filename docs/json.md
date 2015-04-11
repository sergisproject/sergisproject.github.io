---
layout: sergis
title: JSON Format Documentation

extrastyle: 'td > em { white-space: nowrap; } .hide-bullets-in-following-ul + ul { list-style: none; }'

sidebartitle: Table of Contents
sidebar:
- name: SerGIS JSON Object Reference
  href: "#sergis-json-object-reference"
  subitems:
  - name: Condition Object
    href: "#condition-object"
  - name: Action Object
    href: "#action-object"
  - name: Content Object
    href: "#content-object"
  - name: Prompt Object
    href: "#prompt-object"
  - name: Map Object
    href: "#map-object"
- name: SerGIS JSON Game Data
  href: "#sergis-json-game-data"
  subitems:
  - name: Example
    href: "#example"
---
# SerGIS JSON Format Documentation

<p style="text-align: center;"><a href="https://docs.google.com/drawings/d/1aDEHLen7vmv6BJ2mfVee2BMB6y3Zp8NhDK-hQRfoTIc/edit?usp=sharing" target="_blank"><img src="server-client.png" style="border: 1px solid black; padding: 10px; border-radius: 5px;"></a></p>

SerGIS has a special JSON format that is used to store its data. This JSON content is not usually directly available on the client-side, but rather it is used on the server-side so the server can push the data to the client as needed (or directly used by the backend if no server-side system is used).

## SerGIS JSON Object Reference

These objects are referenced below in the JSON spec.

### Condition Object

A SerGIS JSON Condition Object is an object representing a condition that can be resolved to `true` or `false`. It is used in conditional gotos (see [Action Object][actionobject]).

| Property | Type   | Value
| -------- | ----   | -----
| `type`   | string | One of the following: `and`, `or`, `varEmpty`, `varEqualTo`, `varGreaterThan`, `varLessThan`
| `data`   | (varies) | Data that goes along with the condition type.

The following conditions are supported:

| Type | Resolves to true if... | Data Type | Data Description
| ---- | ---------------------- | --------- | ----------------
| `and` | All of its children resolve to true. | array&lt;[Condition][conditionobject]&gt; | One or more other Conditions.
| `or` | At least one of its children resolves to true. | array&lt;[Condition][conditionobject]&gt; | One or more other Conditions.
| `varEmpty` | The specified variable is unset or equal to `0`. | string | The name of the variable to check.
| `varEqualTo` | The specified variable is equal to the specified value. | array `[string, number]` | The name of the variable to check and the value to compare it to.
| `varGreaterThan` | The specified variable is greater than the specified value. | array `[string, number]` | The name of the variable to check and the value to compare it to.
| `varLessThan` | The specified variable is less than the specified value. | array `[string, number]` | The name of the variable to check and the value to compare it to.

### Action Object

A SerGIS JSON Action Object is an object representing either a "Map Action" (an action to do on the map) or a "Gameplay Action" (an action that affects the gameplay).

| Property | Type   | Value
| -------- | -----  | -----
| `name`   | string | The name of the action to perform. Must be either a Gameplay Action (listed below) or a Map Action.
| `frontend` | string | If this is a Map Action, this should be the [frontend][frontends] that the Map Action name is in. (MUST be provided if `name` refers to a Map Action; MUST NOT be provided if `name` refers to a Gameplay Action)
| `data`   | array  | Any data to pass as parameters to the action function. If the action function does not take any parameters, you can leave this out.

**Gameplay Actions:** These actions do not affect the map, but rather affect the game sequence. The `name`s of these actions are:

| Action Name | Data Array Description | Description
| ----------- | ---------------------- | -----------
| `explain` | [[`Content`][contentobject], [`Content`][contentobject], ...] | Show an explanation for why the choice that the user chose was correct or incorrect. The data is an array of [Content objects][contentobject] holding the explanation to display; in most cases, it will be an array of only one [Content object][contentobject]. If this is provided before any Map Actions, it will be shown to the user before those Map Actions are rendered.
| `goto` | [`number|object`] | Go to a specific prompt. **If combined with Map Actions, it must be the *last* action!**
| | | - Simple `goto`: `data`'s only item is a number indicating which prompt index to go to.
| | | - Conditional `goto`: `data`'s only item is an object whose keys are different prompt indexes and whose values are [Condition objects][conditionobject] representing a condition that must be true to go to that prompt index.
| `logout` | (none) | Log the user out. **Cannot be combined with Map Actions!**
| `setVariable` | [`string`, `number`] | Set a numeric variable. The first item in the data array is the name of the variable, and the second is its value.
| `updateVariable` | [`string`, `number`] | Increment or decrement a numeric variable. The first item in the data array is the name of the variable, and the second is the amount to add (can be negative to subtract). If the variable wasn't previously set, it is assumed to be `0`.

**Map Actions:**

The names of map actions vary by [frontend][frontends]. Therefore, in order to support multiple frontends, you must provide a separate Action object for each frontend.

For example, the following could be a list of actions. Note how Gameplay Actions are only specified once, but Map Actions are each specified multiple times, once for each supported frontend.

    [
        {
            "name": "explain",
            "data": [
                {"type": "text", "value": "Choice Explanation Here"}
            ]
        },
        {
            "frontend": "arcgis",
            "name": "buffer",
            "data": [120]
        },
        {
            "frontend": "googlemaps",
            "name": "buffer",
            "data": [120]
        }
    ]

### Content Object

A SerGIS JSON Content Object is an object representing some sort of content that is part of a prompt or choice.

It must have a `type` property that identifies what type of content it is. The value of the `type` property determines which other properties are present.

The main value for the type is stored in `value` (which is required). CSS attributes can be specified by passing them in a string stored in `style` (this should match the formatting of the HTML style attribute). Some types have other possible properties too, all of which are optional (their default values are shown).

 - **Text Type:** `{"type": "text", "value": "plain text here", "centered": false}`
 - **HTML Type:** `{"type": "html", "value": "<p>HTML content here</p>"}`
 - **Image Type:** `{"type": "image", "value": "URL of image", "centered": true}`
 - **YouTube Type:** `{"type": "youtube", "value": "youtube-video-id-here", "width": 400, "height": 300, "playerVars": {"autohide": 1}, "centered": true}`

### Prompt Object

A SerGIS JSON Prompt Object is an object representing either a question for the user or information to show the user.

| Property  | Type   | Value
| --------  | ----   | -----
| `title`   | string | A text-only title for the prompt (usually just the general topic of the question or information).
| `map` (optional) | [Map][mapobject] | A Map object representing the state of the map for this prompt. If any of `latitude`, `longitude`, or `zoom` are not provided in this object, or if `map` is not provided, then the previous values are used. **MUST be provided for the first prompt**, and **should be provided for all prompts if jumping is allowed** (see [Backends](#backends) below).
| `contents` | array&lt;[Content][contentobject]&gt; | The content of the prompt. Each array item must be a Content object. The array must have at least one item.
| `choices` (optional) | array&lt;[Content][contentobject]&gt; | A list of the possible choices for the prompt. Each item must be a Content object that represents the choice. (NOTE: Unlike in the `contents` property, only one Content object can be provided for each choice.) If not provided, or if empty, a "Continue" button is shown if it is not the last prompt. (This may be useful if the prompt just provides information instead of asking a question.)
| `randomizeChoices` (optional) | boolean | Whether to randomize the choices for this prompt.
| `buttons` (optional) | object | Frontend-specific toolbar buttons to show/hide/enable/disable. This should be an object whose keys are frontend names and whose values are another object. This object should then have keys matching button IDs and values being another object with `hidden` or `disabled` properties (to change the state of the button with that ID).

Example value for `buttons`:

    "buttons": {
        "arcgis": {
            "measureButton": {
                "hidden": false,
                "disabled": true
            }
        },
        "googlemaps": {
            "measureButton": {
                "hidden": false,
                "disabled": true
            }
        }
    }

### Map Object

A SerGIS JSON Map Object is an object representing a map state, including location (i.e. latitude/longitude) and zoom.

| Property | Type   | Value
| -------- | ------ | -----
| `latitude` | number | The latitude position (negative values are north, positive are south).
| `longitude` | number | The longitude position (negative values are west, positive are east).
| `zoom` | number | The zoom level of the map.
| `frontendInfo` | object | An object with frontend-specific map information, where each key is the name of a [frontends][frontends] (corresponding to the frontend's name property) and the value is an object with specific information for that frontend. To find out the values for each frontend, look at the top of the frontend's file for a comment block starting with "SerGIS JSON Map Object - frontendInfo for [frontend name]".

## SerGIS JSON Game Data

SerGIS JSON Game Data is a JSON file with a specific structure. The JSON data consists of an object with the following properties:

| Property | Type | Value
| -------- | ---- | -----
| `layout` (optional) | object | Configuration regarding the layout of the game (see below).
| `jumpingBackAllowed` (optional) | boolean | Whether the user is allowed to go back to previously answered prompts, allowing the user to change the values that he or she put. (Provided to the client through the `logIn` and `getUser` functions of the [client backend][backends].) Default: `false`
| `onJumpBack` (optional) | string | Says what should happen regarding prompts (after the one to which the user is jumping back) for which the user already made a choice. One of the following: `"reset"` (disregard all the choices that the user has made on prompts after the one he or she is jumping back to), `"hide"` (remember the user's choices, but don't show any Map Actions on the map), Anything else (e.g., an empty string, or just not providing `onJumpBack`) - remember the user's choices and show the corresponding Map Actions on the map
| `jumpingForwardAllowed` (optional) | boolean | Whether the user is allowed to skip prompts and come back to them later. If this is `true` but `jumpingBackAllowed` is not, then the user will not be able to go back to questions that he or she skips. (Provided to the client through the `logIn` and `getUser` functions of the [client backend][backends].) Default: `false`
| `showActionsInUserOrder` (optional) | boolean | Whether to render the Map Actions in the order that the user went through the prompts (applies if `jumpingForwardAllowed` and/or `jumpingBackAllowed` are true). If this is false, the actions are rendered in the order of the prompts that they come from, regardless of the order in which the user chose them. Should be utilized by the handler for the `getPreviousMapActions` function of the [client backend][backends]. Default: `false`
| `promptList` | array | An array of objects representing the different prompts and choices (see below).

- The `layout` object has the following properties:

  | Property | Type | Value
  | -------- | ---- | -----
  | `defaultSidebarWidthRatio` | number | A number between 0 and 1 indicating the default % of the horizontal screen real estate that should be taken up by the prompt sidebar. Default: ???
  | `disableSidebarResizing` | boolean | Whether horizontal resizing of the prompt sidebar should be disabled. Default: `false`
  | `disableTranslucentSidebar` | boolean | Whether the translucent prompt sidebar, with the map behind it, should be opaque instead, with the map only extending to its border and not behind it. Default: `false`
  | `showPromptNumber` | boolean | Whether to show "Prompt __ of __" at the bottom of the prompt sidebar. (If any kind of jumping around is enabled, then this is always shown regardless of this setting.) Default: `false`

- Each object in the `promptList` array has the following properties:

  | Property | Type | Value
  | -------- | ---- | -----
  | `prompt` | [Prompt][promptobject] | The SerGIS Prompt object representing the prompt.
  | `actionList` (optional) | array | An array of objects representing different actions. Each item in this array corresponds to a choice in the `prompt.choices` array. If `prompt.choices` is empty or not provided, then this can be empty or not provided. (This is separate from `prompt` so a server can send `prompt` on to the client without revealing which choice is best.)
  
  <div class="hide-bullets-in-following-ul" style="display: none;"></div>

  - Each object in the `actionList` array has the following properties:

    | Property | Type | Value
    | -------- | ---- | -----
    | `actions` (optional) | array<[Action][actionobject]> | An array of SerGIS Action objects representing the actions to be taken if this choice is selected. (Actions are evaluated in the order that they appear in this array.) After these actions are taken, or if no actions are provided (i.e. `actions` is an empty array), the game will advance to the next prompt automatically (unless otherwise instructed).
    | `pointValue` (optional) | number | The amount of points that the user should have added to his score for choosing this choice. If not provided, defaults to `0`.
    
    NOTE: If neither of these properties are specified, `actionList` should still contain **an empty object** (`{}`) filling the position.

### Example

An example can be seen in the [sergis-client repository](https://github.com/sergisproject/sergis-client), in [the testdata.json file](https://github.com/sergisproject/sergis-client/blob/master/lib/testdata.json) (or see an [older, commented version](https://github.com/sergisproject/sergis-client/blob/d030e08bfe084f669cd29225a55586ac0aebb5b7/testdata.js)).



[conditionobject]:  json.html#condition-object  "SerGIS JSON Condition Object"
[actionobject]:  json.html#action-object  "SerGIS JSON Action Object"
[contentobject]: json.html#content-object "SerGIS JSON Content Object"
[promptobject]:  json.html#prompt-object  "SerGIS JSON Prompt Object"
[mapobject]:     json.html#map-object     "SerGIS JSON Map Object"

[sergis-json-game-data]: json.html#sergis-json-game-data "SerGIS JSON Game Data"

[frontends]: client.html#frontends "SerGIS Client Frontends"
[backends]:  client.html#backends  "SerGIS Client Backends"

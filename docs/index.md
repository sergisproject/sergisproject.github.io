---
layout: sergis
title: Documentation
---
# Documentation

<p style="text-align: center;"><img src="server-client.png" style="border: 1px solid black; padding: 10px; border-radius: 5px;"></p>

## SerGIS Object Reference

### Action Object

An object representing an action to do on the map.

 - `name` (string) The name of the action to perform (corresponding to a function in the `sergis.actions` object, defined in `lib/actions.js`).
 - `data` (array) Any data to pass as parameters to the action function.

### Question Object

An object representing a question for the user.

 - `title` (string) A text-only title for the question.
 - `content` (string) An HTML description for the question.
 - `answers` (array<string>) A list of the possible answers for the question; each is represented by a string (HTML).

## Backends

In SerGIS, a backend is a JavaScript library for the SerGIS client that handles the interaction between the server and the client UI.

Each backends is a JavaScript object with the following properties.

**Any property that is a function should return a *JavaScript Promise* that is resolved with the value in question (or resolved with no data if the function does not return a value). In the case of an error, the function should reject the promise with an error message.**

 - `account`: an object with the following properties:
   - `logIn(username, password)` (returns Promise<string>) A function to attempt to log in with the provided username and password. If successful, it should resolve the promise with the display name.
   - `logOut()` (returns Promise) A function to log the user out.
   - `getUser()` (returns Promise<string>) A function to get the user's display name.
 - `game`: an object with the following properties:
   - `allowJumpingAround` A boolean that, if true, allows the user to skip around different questions. (If false, the user will only be allowed to proceed through questions in a forward, sequential order.)
   - `getPreviousActions()` (returns Promise<array&lt;Action&gt;>) A function to get a list of all the previous actions that the user has chosen up to this point (used if the SerGIS UI has to re-draw the actions on the map).
   - `getQuestionCount()` (returns Promise&lt;number&gt;) A function to get the total number of questions.
   - `getQuestion(questionIndex)` (returns Promise&lt;Question&gt;) A function to go to a question number and returns the Question object representing the question. (Make sure to check on the server if the user has permission to go to this question; even if allowJumpingAround is false, anything on the client side of things can be manipulated.) Also, this function should save the current state on the server (i.e. which question the user is on) so the user can resume where he or she left off.
   - `getAction(questionIndex, answerIndex)` (returns Promise&lt;Action&gt;) A function to get the action for a specific question number and answer number within that question.

---
layout: sergis
---
# Documentation

<p><img src="server-client.png"></p>

## SerGIS Object Reference

### Action Object

An object representing an action.

### Question Object

An object representing a question.

 - `title` (string) A text-only title for the question.
 - `content` (string) An HTML description for the question.
 - `answers` (array<string>) A list of the possible answers for the question; each is represented by a string (HTML).

## Backends

In SerGIS, a backend is a JavaScript library for the SerGIS client that handles the interaction between the server and the client UI.

Each backends is a JavaScript object with the following properties:

 - `account`: an object with the following properties:
   - `logIn(username, password)` (returns Promise<string>) A function to attempt to log in with the provided username and password. If successful, it should resolve the promise with the display name. If unsuccessful, it should reject the promise with an error message.
   - `logOut()` (returns Promise) A function to log the user out.
   - `getUser()` (returns Promise<string>) A function to get the user's display name. If successful, it should resolve the promise with the display name. If unsuccessful, it should reject the promise with an error message.
 - `game`: an object with the following properties:
   - `getPreviousActions()` (returns Promise<array<Action>>) A function to get a list of all the previous actions that the user has chosen up to this point (used if the SerGIS UI has to re-draw the actions on the map).
   - `allowJumpingAround` A boolean that, if true, allows the user to skip around different questions. (If false, the user will only be allowed to proceed through questions in a forward, sequential order.)
   - `goto(question)` (returns Promise<Question>) A function to go to a question number and returns the Question object representing the question. (Make sure to check on the server if the user has permission to go to this question; even if allowJumpingAround is false, anything on the client side of things can be manipulated.) Also, this function should save the current state on the server (i.e. which question the user is on) so the user can resume where he or she left off.
   - `getAction(question, answer)` (returns Promise<Action>) A function to get the action for a specific question number and answer number within that question.

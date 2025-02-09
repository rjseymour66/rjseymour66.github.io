---
title: "Location, navigation, and history"
linkTitle: "Location, nav, history"
weight: 50
description:
---

## Location

The Location object is represented by the `location` property on the Window and Document object, and it represents the current URL of the document displayed in the window:
- `window.location` is the same as `document.location`
- `document.URL` is the URL string
- provides an API for loading new documents in the window
- `href` property and `toString()` method return the URL
- `hash` returns the fragment identifier
- `search` returns the part of the URL that starts with a `?` - the query string

`URL` objects have a `searchParams` property that is a parsed representation of the `search` property
- Location object does not have the `searchParams` property - you need to make a URL object out of the location object and then you can parse the query parameters:

```js
// http://127.0.0.1:5500/index.html?q=test
let url = new URL(window.location);         // URL object  
let query = url.searchParams.get("q");      // test
```

### Loading new documents

You can change the document that the browser loads by assigning `window.location` a new URL string or fragment:
- `window.location = 'http://www.google.com';` - absolute
- `window.location = 'headings.html';` - relative
- `location = '#section1';` - fragment
- `replace()` method takes a string URL and loads a new page, and then also replaces the calling document in the browser's history.
  - Use case: if the user's browser doesn' not support features on your JS page, you can use `replace('https://static-site.com')` to replace it with a static version.
- `reload()` method makes the document reload

## Browsing history

The History object represents the history for the window:
- comes from early days of the web when everything happened on the server
  - modern web pages load content dynamically, so they are not always loading pages from the server
  - these applications manage history - really viewing previous app states - with hashchange events or `pushState()`
- browsing history is a list of documents and document states
- length is stored in `list` property
- scripts are not allowed access to history URLs, for security
- history object methods:
  - `back()`: go to previous page in history
  - `forward()`: go to previous page in history
  - `go()`: takes a positive or negative integer to jump that many pages forward or backward
- child window elements like `<iframe>` navigate forwards or backwards with the main window

```js
history.back()
history.forward()
history.go(-3)      // press back button 3x
history.go(0)       // reload current page
```

### hashchange Events

hashchange events use the `location.hash` property:
- `location.hash` takes an arbitrary fragment identifier - just don't pick a name of an existing element ID so there is no chance it will scroll to the fragment
  - create a unique fragment identifier to for each state of your application to let the user use browsing history
  - setting the property updates the URL and adds an entry into the browser history
  - when you set the fragment explicitly or when it changes, a 'hashchange' event is fired on the Window object to notify you that the user is using browsing history
- This method is hacky


#### Steps:
1. Define a fragment - encode the state information that your app needs to render the page into a short string of text
2. Write a function that convers page state into a string
3. Write a functino that parses the string to re-create the page state it represents
4. Write `window.onhashchange` or `window.addEventListener('hashchange', ()=>{})` that reads the fragment in `location.hash` and converts that string into a representation of that state
5. When the user initiates a new state, don't render it directly - encode the state as a string and set `location.hash` to that string. This triggers the hashchange event and the handler will display the new state  

### pushState()

`history.pushState()` uses a 'popstate' event to track history:
- adds an object that represents the state to the browser history
- when the user clicks forward or back, it fires a 'popstate' event with a copy of the saved state object and the app recreates the state
  
- Also saves a URL for each state, which means users can bookmark states

Object arguments:
- First arg: the object - supports Map, Set, Date, and typed arrays with ArrayBuffers - it is serialized with the structured clone algorithm, which is more robust than `JSON.stringify()`
  - The event object for the `popstate` event has a `state` property that contains a copy of the object that you pass
- Second arg: string that was supposed to be title for state, but its not generally supported, so pass empty string
- Third arg: optional, a URL displayed in location bar immediately and if the user returns to this state with browser back or forward button
  - users can bookmark, but you have to restore the state by parsing the URL

### replaceState()

Takes same args as `pushState()` but replaces current history state instead of adding a new state to the browser history:
- when you load an app that uses `pushState()`, you should call `replaceState()` at the start to define the initial app state

## Example program

This is a number guessing program that demonstrates `pushState()` and `replaceState()`:

### JS

```js
class GameState {
    // factory function
    static newGame() {
        let s = new GameState();
        s.secret = s.randomInt(0, 100);  // 0 < n < 100
        s.low = 0;                      // guesses must be greater than this
        s.high = 100;                   // guesses must be lower than this
        s.numGuesses = 0;               // num of guesses already made
        s.guess = null;                 // what the last guess was
        return s;
    }

    // recreate GameState object based on plain object that we get from 
    // popstate event
    static fromStateObject(stateObject) {
        let s = new GameState();
        for (let key of Object.keys(stateObject)) {
            s[key] = stateObject[key];
        }
        return s;
    }

    // encode the state of any game as a URL.
    toURL() {
        let url = new URL(window.location);
        url.searchParams.set("l", this.low);
        url.searchParams.set("h", this.high);
        url.searchParams.set("n", this.numGuesses);
        url.searchParams.set("g", this.guess);
        return url.href;
    }

    // factory func that creates a new GameState obj and initializes it
    // with a URL. returns null if there are errors in the parameters
    static fromURL(url) {
        let s = new GameState();
        let params = new URL(url).searchParams;
        s.low = parseInt(params.get('l'));
        s.high = parseInt(params.get('h'));
        s.numGuesses = parseInt(params.get('n'));
        s.guess = parseInt(params.get('g'));

        // return null if URL is missing params
        if (isNaN(s.low) || isNaN(s.high) ||
            isNaN(s.numGuesses) || isNaN(s.guess)) {
            return null;
        }

        // pick new secret number when you restore the game from URL
        s.secret = s.randomInt(s.low, s.high);
        return s;
    }

    // return integer n, min < n < max
    randomInt(min, max) {
        return min + Math.ceil(Math.random() * (max - min - 1));
    }

    // modify the document to display current state of the game
    render() {
        let heading = document.querySelector('#heading');
        let range = document.querySelector('#range');
        let input = document.querySelector('#input');
        let playagain = document.querySelector('#playagain');

        // update the document heading and title
        heading.textContent = document.title = `I'm thinking of a number between ${this.low} and ${this.high}.`;

        // update the visual range of numbers
        range.style.marginLeft = `${this.low}%`;
        range.style.width = `${this.high - this.low}%`;

        // make sure the input field is empty and focused
        input.value = "";
        input.focus();

        // display feedback based on user's last guess
        if (this.guess === null) {
            input.placeholder = 'Type your guess and hit Enter';
        } else if (this.guess < this.secret) {
            input.placehoder = `${this.guess} is too low. Guess again.`;
        } else if (this.guess > this.secret) {
            input.placehoder = `${this.guess} is too high. Guess again.`;
        } else {
            input.placeholder = document.title = `${this.guess} is correct!`;
            heading.textContent = `You win in ${this.numGuesses} guesses!`;
            playagain.hidden = false;
        }
    }

    // Update the state of the game based on what the user guessed.
    updateForGuess(guess) {
        // If it is a number and is in the right range
        if ((guess > this.low) && (guess < this.high)) {
            // update state obj based on guess
            if (guess < this.secret) this.low = guess;
            else if (guess > this.secret) this.high = guess;
            this.guess = guess;
            this.numGuesses++;
            return true;
        } else {
            // invalid guess: notify user but don't update state
            alert(`Please enter a number greater than ${this.low} and less than ${this.high}`);
            return false;
        }
    }
}

// Initialize, update, save, and render the state obj when appropriate

// either load existig game from URL or start new game
let gamestate = GameState.fromURL(window.location) || GameState.newGame();

// save initial state with replaceState
history.replaceState(gamestate, gamestate.toURL());

// display initial state
gamestate.render();

// after user guess, update state then save new state to browser history
// and render new state
document.querySelector('#input').onchange = (event) => {
    if (gamestate.updateForGuess(parseInt(event.target.value))) {
        history.pushState(gamestate, "", gamestate.toURL());
    }
    gamestate.render();
};

// if users goes back or forward in history, you get popstate event
// on window obj w a copy of the state obj saved in pushState.
// When this happens, render the new state
window.onpopstate = event => {
    gamestate = GameState.fromStateObject(event.state);
    gamestate.render();
};
```

### HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script defer src="pushState.js"></script>
    <title>Location and history</title>
    <style>
        body {
            height: 250px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-evenly;
        }

        #heading {
            font: bold 36px sans-serif;
            margin: 0;
        }

        #container {
            border: solid black 1px;
            height: 1em;
            width: 80%;
        }

        #range {
            background-color: green;
            margin-left: 0%;
            height: 1em;
            width: 100%;
        }

        #input {
            display: block;
            font-size: 24px;
            width: 60%;
            padding: 5px;
        }

        #playagain {
            font-size: 24px;
            padding: 10px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1 id="heading">I'm thinking of a number...</h1>
    <!-- visual representatino of the numbers that are not yet ruled out -->
    <div class="container">
        <div id="range"></div>
    </div>
    <!-- where user enters guess -->
    <input type="text" name="name" id="input">
    <!-- button that reloads with no search string. Hidden until game ends -->
    <button id="playagain" hidden onclick="location.search='';">Play Again</button>


</body>
</html>
```

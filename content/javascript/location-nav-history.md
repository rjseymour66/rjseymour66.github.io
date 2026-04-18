---
title: "Location, navigation, and history"
linkTitle: "Location, nav, history"
weight: 80
description:
---

## Location

The *Location* object, accessible as `window.location` or `document.location`, represents the current URL of the document displayed in the window and provides an API for loading new documents. The `href` property and `toString()` method return the full URL string. The `hash` property returns the fragment identifier, and `search` returns the query string, which is the portion of the URL starting with `?`. The `document.URL` property also returns the URL as a plain string.

`URL` objects have a `searchParams` property that is a parsed representation of the `search` property. The `Location` object does not have `searchParams`, so to parse query parameters you need to construct a `URL` object from `window.location`:

```js
// http://127.0.0.1:5500/index.html?q=test
let url = new URL(window.location);         // URL object  
let query = url.searchParams.get("q");      // test
```

### Real-world example: shareable filter state

Store UI state (current page, active filters, sort order) in query parameters so users can bookmark and share filtered views. Update the URL without reloading the page using `history.pushState`:

```js
// Read state from URL on page load
function getFilters() {
    const params = new URL(window.location).searchParams;
    return {
        page:     parseInt(params.get('page') ?? '1', 10),
        category: params.get('category') ?? 'all',
        sort:     params.get('sort') ?? 'newest',
    };
}

// Apply a filter and update the URL
function applyFilter(key, value) {
    const url = new URL(window.location);
    url.searchParams.set(key, value);
    url.searchParams.set('page', '1');   // reset to page 1 on filter change
    history.pushState({}, '', url);
    renderResults(getFilters());
}

// Initialize on load
renderResults(getFilters());

// Re-render on back/forward navigation
window.addEventListener('popstate', () => renderResults(getFilters()));
```

A URL like `/products?category=shoes&sort=price&page=2` is now shareable and bookmarkable.

### Loading new documents

You can load a new document by assigning a URL string to `window.location`. Assign an absolute URL, a relative URL, or a fragment identifier:

```js
window.location = 'http://www.google.com'; // absolute URL
window.location = 'headings.html';          // relative URL
location = '#section1';                     // fragment identifier
```

The `replace()` method loads a new page and replaces the calling document in the browser history, so it does not add a new entry. If the user's browser does not support features on your page, you can call `replace('https://static-site.com')` to redirect to a static version. The `reload()` method reloads the current document.

## Browsing history

The *History* object represents the browsing history for the window. It originates from the early days of the web, when every navigation loaded a new document from the server. Modern web pages load content dynamically without full page reloads, so applications manage history by tracking application states rather than server-fetched documents. They do this using hashchange events or `pushState()`.

Browsing history is a list of documents and document states. The `length` property stores the number of entries. Scripts cannot access the URLs in the history list for security reasons. Child window elements such as `<iframe>` navigate forward or backward alongside the main window.

The History object provides three navigation methods:

- **`back()`:** Go to the previous page in history
- **`forward()`:** Go to the next page in history
- **`go()`:** Takes a positive or negative integer to jump that many pages forward or backward

```js
history.back()
history.forward()
history.go(-3)      // press back button 3x
history.go(0)       // reload current page
```

### hashchange events

hashchange events rely on the `location.hash` property. Set `location.hash` to an arbitrary fragment identifier to represent the current application state. Avoid reusing an existing element ID so the browser does not scroll to that element. Each time you set the property, the browser updates the URL and adds an entry to the browser history. When the fragment changes, a `hashchange` event fires on the `Window` object, notifying you that the user has navigated.

This approach is considered a legacy technique, but it is simpler to implement than `pushState()` and works in all browsers.

#### Steps

1. Define a fragment: encode the state information your app needs to render the page into a short string.
2. Write a function that converts page state into a string.
3. Write a function that parses the string to re-create the page state it represents.
4. Write a `hashchange` handler that reads `location.hash` and converts the string back into application state:

```js
window.addEventListener('hashchange', () => {
    const state = parseState(location.hash.slice(1));
    renderState(state);
});
```

5. When the user initiates a new state, do not render it directly. Encode the state as a string and set `location.hash` to that string. This triggers the `hashchange` event and the handler displays the new state.

### pushState()

`history.pushState()` uses a `popstate` event to track history. It adds an object representing the current application state to the browser history. When the user clicks the forward or back button, the browser fires a `popstate` event with a copy of the saved state object, and the application recreates the state. Unlike hashchange, `pushState()` also saves a URL for each state, so users can bookmark individual states.

`pushState()` accepts three arguments:

- **First argument:** The state object. Supports `Map`, `Set`, `Date`, and typed arrays with `ArrayBuffer`. The browser serializes it with the structured clone algorithm, which is more robust than `JSON.stringify()`. The `popstate` event object exposes a copy of this object through its `state` property.
- **Second argument:** A title string. It was intended to name the state, but browsers do not generally support it. Pass an empty string.
- **Third argument (optional):** A URL displayed in the location bar immediately and when the user returns to this state with the back or forward button. Users can bookmark it, but you must restore the state by parsing the URL.

### replaceState()

`replaceState()` takes the same arguments as `pushState()` but replaces the current history entry instead of adding a new one. Call `replaceState()` when the application first loads to define the initial state, so the user's first back-button press does not land on a blank entry.

### Real-world example: minimal client-side router

Single-page applications intercept link clicks and use `pushState` to swap views without reloading. This is the core of every frontend router (React Router, Vue Router, etc.):

```js
// Define routes as path → render function pairs
const routes = {
    '/':        renderHome,
    '/products': renderProducts,
    '/about':   renderAbout,
};

function renderNotFound() {
    document.querySelector('#app').innerHTML = '<h1>404 - Page not found</h1>';
}

function navigate(path) {
    history.pushState({}, '', path);
    render(path);
}

function render(path) {
    const handler = routes[path] ?? renderNotFound;
    handler();
}

// Intercept clicks on internal links — prevent full page reload
document.addEventListener('click', (e) => {
    const link = e.target.closest('[data-link]');
    if (!link) return;

    e.preventDefault();
    navigate(link.getAttribute('href'));
});

// Handle browser back/forward buttons
window.addEventListener('popstate', () => {
    render(window.location.pathname);
});

// Render the initial view
render(window.location.pathname);
```

Mark internal links with `data-link` to distinguish them from external links that should reload normally:

```html
<nav>
  <a href="/" data-link>Home</a>
  <a href="/products" data-link>Products</a>
  <a href="/about" data-link>About</a>
  <a href="https://example.com">External - no data-link, reloads</a>
</nav>
<main id="app"></main>
```

## Example program

The following program implements a number guessing game that demonstrates `pushState()` and `replaceState()`:

### JavaScript

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
            input.placeholder = `${this.guess} is too low. Guess again.`;
        } else if (this.guess > this.secret) {
            input.placeholder = `${this.guess} is too high. Guess again.`;
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

// either load existing game from URL or start new game
let gamestate = GameState.fromURL(window.location) || GameState.newGame();

// save initial state with replaceState
history.replaceState(gamestate, "", gamestate.toURL());

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
    <!-- visual representation of the numbers that are not yet ruled out -->
    <div id="container">
        <div id="range"></div>
    </div>
    <!-- where user enters guess -->
    <input type="text" name="name" id="input">
    <!-- button that reloads with no search string. Hidden until game ends -->
    <button id="playagain" hidden onclick="location.search='';">Play Again</button>


</body>
</html>
```

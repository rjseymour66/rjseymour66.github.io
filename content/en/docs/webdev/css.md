---
title: "CSS"
# linkTitle: "CSS"
weight: 15
description: >
  Setting up a CSS file.
---

## Reset

Add this reset to your file (adapted from [A Modern CSS Reset](https://andy-bell.co.uk/a-modern-css-reset/)):

```css
/* Box sizing rules */
*,
*::before,
*::after {
  box-sizing: border-box;
}

/* Remove default margin */
* {
  margin: 0;
  padding: 0;
  /* inherit font so you can define styles 
  for h1, h* elements instead of rely on user 
  agent defaults */
  font: inherit;
}

/* Remove list styles on ul, ol elements with a list role, which suggests default styling will be removed */
ul[role="list"],
ol[role="list"] {
  list-style: none;
}

/* Set core root defaults */
html:focus-within {
  scroll-behavior: smooth;
}

html,
body {
  height: 100%;
}

/* Set core body defaults */
body {
  text-rendering: optimizeSpeed;
  line-height: 1.5;
}

/* A elements that don't have a class get default styles */
a:not([class]) {
  text-decoration-skip-ink: auto;
}

/* Make images easier to work with */
img,
picture,
svg {
  max-width: 100%;
  display: block;
}

/* Remove all animations, transitions and smooth scroll for people that prefer not to see them */
@media (prefers-reduced-motion: reduce) {
  html:focus-within {
    scroll-behavior: auto;
  }

  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## Set up variables

This example shows you how to set up variables for the entire site:

```css
:root {
  --clr-accent-500: hsl(12, 60%, 45%);
  --clr-accent-400: hsl(12, 88%, 59%);
  --clr-accent-300: hsl(12, 88%, 75%);
  --clr-accent-100: hsl(13, 100%, 96%);

  --clr-primary-400: hsl(228, 39%, 23%);

  --clr-neutral-900: hsl(232, 12%, 13%);
  --clr-neutral-100: hsl(0 0% 100%);

  --ff-primary: "Be Vietnam Pro", sans-serif;

  --ff-body: var(--ff-primary);
  --ff-heading: var(--ff-primary);

  --fw-regular: 400;
  --fw-semi-bold: 500;
  --fw-bold: 700;

  --fs-300: 0.8125rem;
  --fs-400: 0.875rem;
  --fs-500: 0.9375rem;
  --fs-600: 1rem;
  --fs-700: 1.875rem;
  --fs-800: 2.5rem;
  --fs-900: 3.5rem;

  --fs-body: var(--fs-400);
  --fs-primary-heading: var(--fs-800);
  --fs-secondary-heading: var(--fs-700);
  --fs-nav: var(--fs-500);
  --fs-button: var(--fs-300);

  --size-100: 0.25rem;
  --size-200: 0.5rem;
  --size-300: 0.75rem;
  --size-400: 1rem;
  --size-500: 1.5rem;
  --size-600: 2rem;
  --size-700: 3rem;
  --size-800: 4rem;
  --size-900: 5rem;
}

@media (min-width: 50em) {
  :root {
    --fs-body: var(--fs-500);
    --fs-primary-heading: var(--fs-900);
    --fs-secondary-heading: var(--fs-800);

    --fs-nav: var(--fs-300);
  }
}
```

## Forms

```html
<body>
   <h1>Registration Form</h1>
   <p>Please fill out this form with the required information</p>

   <form action="https://register-demo.freecodecamp.org" method="post">
      <fieldset>
         <label for="first-name">Enter Your First Name: <input id="first-name" type="text" name="first-name"
               required /></label>
         <label for="last-name">Enter Your Last Name: <input id="last-name" type="text" name="last-name"
               required /></label>
         <label for="email">Enter Your Email: <input id="email" type="email" name="email" required /></label>
         <label for="new-password">Create a New Password: <input id="new-password" type="password"
               pattern="[a-z0-5]{8,}" name="new-password" required /></label>
      </fieldset>
      <fieldset>
         <label for="personal-account"><input class="inline" id="personal-account" type="radio" name="account-type" /> Personal
            Account</label>
         <label for="business-account"><input class="inline" id="business-account" type="radio" name="account-type" /> Business
            Account</label>
         <label for="terms-and-conditions">
            <input class="inline" id="terms-and-conditions" type="checkbox" name="terms-and-conditions" required /> I accept the <a
               href="https://www.freecodecamp.org/news/terms-of-service/">terms and conditions</a>
         </label>
      </fieldset>
      <fieldset>
         <label for="profile-picture">Upload a profile picture: <input id="profile-picture" type="file"
               name="profile-picture" /></label>
         <label for="age">Input your age (years): <input id="age" type="number" min="13" max="120" name="age" /></label>
         <label for="referrer">How did you hear about us?
            <select id="referrer" name="referrer">
               <option value="">(select one)</option>
               <option value="1">freeCodeCamp News</option>
               <option value="2">freeCodeCamp YouTube Channel</option>
               <option value="3">freeCodeCamp Forum</option>
               <option value="4">Other</option>
            </select>
         </label>
         <label for="bio">Provide a bio:
            <textarea id="bio" rows="3" cols="30" name="bio" placeholder="I like coding on the beach..."></textarea>
         </label>
      </fieldset>

      <!-- The first input element with a type of submit is automatically set to submit its nearest parent form element. -->
      <input type="submit" value="Submit" />
   </form>

</body>
```

The related CSS stylesheet:

```css
body {
   width: 100%;
   height: 100vh;
   margin: 0;
   background-color: #1b1b32;
   color: #f5f6f7;
   font-family: Tahoma, Geneva, Verdana, sans-serif;
   font-size: 16px;
 }

label {
   display: block;
   margin: 0.5rem 0;
}

h1,
p {
   margin: 1em auto;
   text-align: center;
}

form {
   margin: 0 auto;
   max-width: 500px;
   min-width: 300px;
   width: 60vw;
   padding: 0 0 2em 0;
}

fieldset {
   border: none;
   padding: 2rem 0;
   border-bottom: 3px solid #3b3b4f;
}

fieldset:last-of-type {
   border-bottom: none;
}

/* 
fieldset:not(:last-of-type) {
   border-bottom: 3px solid #3b3b4f;
} */

input,
textarea,
select {
   width: 100%;
   margin: 10px 0 0 0;
   min-height: 2em;
}

.inline {
   /* unsets the 100% width */
   width: unset;
   margin: 0 0.5em 0 0;
   vertical-align: middle;
}

input,
textarea {
   background-color: #0a0a23;
   border: 1px solid #0a0a23;
   color: #fff;
   
}

input[type="submit"] {
   display: block;
   width: 60%;
   margin: 1em auto;
   height: 2em;
   min-width: 300px;
   font-size: 1.1rem;
   background-color: #3b3b4f;
   border-color: white;
}

input[type="file"] {
   padding: 1px 2px;
}

a {
   color: #dfdfe2;
}
```
---
title: "Sample styling sheet"
linkTitle: "Sample styling sheet"
weight: 301
description: >
  Sample styling sheet.
---

> Deprecated
> Still has some good info, so keeping.

```css
/* -------------------- */
/* Helpful links        */
/* -------------------- */

/* clamp details: https://css-tricks.com/linearly-scale-font-size-with-css-clamp-based-on-the-viewport/ */
/* breakpoints:   https://www.freecodecamp.org/news/the-100-correct-way-to-do-css-breakpoints-88d6a5ba1862/
                  600px, 900px, 1200px, and 1800px
*/



/* -------------------- */
/* Custom Properties    */
/* -------------------- */

:root {
    /* colors */
    --clr-dark: 230 35% 7%;

    /* font sizes */
    /* 400 is the base size. Work from base 16 */
    --fs-900: clamp(5rem, 8vw + 1rem, 9.375rem);
    --fs-400: 0.9375rem;

    /* font-families */
    --ff-serif: 'Bellefair', serif;

    /* use em because of Safari issue */
    @media (min-width: 35em) {

    }

    @media (min-width: 45em) {

    }
}

/* -------------------- */
/* Reset                */
/* -------------------- */

/* https://piccalil.li/blog/a-modern-css-reset/ */

/* Box sizing */
*,
*::before,
*::after {
    box-sizing: border-box;
}

/* Reset margins (collapsing margins) */
body,
h1,
h2,
h3,
h4,
h5,
p,
figure,
picture {
    margin: 0;
}

h1,
h2,
h3,
h4,
h5,
h6,
p {
    font-weight: 400;
}

/* set up the body */
body {
    font-family: var(--ff-sans-normal);
    font-size: var(--fs-400);
    color: hsl(var(--clr-white));
    background-color: hsl(var(--clr-dark));
    line-height: 1.5;  /* browser default is 1.4 */
    min-height: 100vh; /* prevents strange short pages */
    
    display: grid;
    grid-template-rows: min-content 1fr;

    /*  */
    overflow-x: hidden;
}

/* make images easier to work with */
img,
picture {
    max-width: 100%;
    display: block;
}

/* make forms easier to work with.
By default, these items do not inherit font properties */
input,
button,
textarea,
select {
    font: inherit;
}

/* remove animations for people who've turned them off (when motion causes problems) */
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
    }
}

/* -------------------- */
/* Utility Classes      */
/* -------------------- */

/* gap is custom prop you can set on each flex/grid container */
.flex {
    display: flex;
    gap: var(--gap, 1rem);
}

/* https://css-tricks.com/snippets/css/complete-guide-grid/ */
.grid {
    display: grid;
    gap: var(--gap, 1rem);
}

.d-block {
    display: block;
}

/* --flow-space lets you set the margin-top differently on specific els */


.flow  * + * {
    margin-top: var(--flow-space, 1rem);
}

.flow--space-small {
    --flow-space: .5rem;
}

.container {
    padding: 0 2em;
    margin: 0 auto;
    max-width: 80rem;
    /* margin-inline: auto; */
    /* padding-inline: 2em (l/r not t/b) */
}

/* screen reader only. Still in DOM, just hidden */
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0,0,0,0);
    white-space: nowrap;
    border: 0;
}

.skip-to-content {
    /* absolute pulls it out of the flow so it doesn't interfere with other elements */
    position: absolute;
    z-index: 9999;
    background: hsl(var( --clr-white));
    color: hsl(var( --clr-dark));
    padding: .5em 1em;
    margin-inline: auto;
    /* hides this from the screen */
    transform: translateY(-100%);
}

.skip-to-content:focus {
    transform: translateY(0);
    transition: transform 250ms ease-in-out;
}

/* colors */

.bg-dark   { background-color: hsl( var(--clr-dark) );}

/* typography */

.ff-serif { font-family: var(--ff-serif); }

/* you can use px for small and specific things every now and then */
.letter-spacing-1 { letter-spacing: 4.75px; }
.letter-spacing-2 { letter-spacing: 2.7px; }

.uppercase { text-transform: uppercase; }

.fs-900 { font-size: var(--fs-900); }
.fs-400 { font-size: var(--fs-400); }

.fs-900,
.fs-800,
.fs-700,
.fs-600 {
    line-height: 1.1;
}

/* -------------------- */
/* Components           */
/* -------------------- */

/* primary header */



/* --------------------------- *\
/* Layout                      *\
/* --------------------------- */



.grid-container {
    text-align: center;
    display: grid;
    place-items: center;
    padding-inline: 1rem;
    padding-bottom: 4rem;
}

@media (min-width: 45em) {
    .grid-container {
        text-align: left;
        column-gap: var(--container-gap, 2rem);
        /* 2rem -> as much space as you need | 0-40rem | 0-40rem | 2rem -> as much space as you need */
        grid-template-columns: minmax(2rem, 1fr) repeat(2, minmax(0, 40rem)) minmax(2rem, 1fr);
    }
```
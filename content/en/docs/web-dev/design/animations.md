---
title: "Enhancing layouts with animation"
linkTitle: "Animations"
weight: 60
# description:
---

You should only use animation when appropriate and when it improves usability:
- combine it with design fundamentals like contrast to provide obvious cues about where users should go next
- demonstrate how to use the site or indicate the function of something
- animations should add, not subtract from the site
- elicits an emotional response. for example, putting christmas lights on a site around the holidays

When animations slow down the UI, its called 'jank':
- chagning the size of elements means that the browser needs to recalculate and repaint the page
- see [CSS Triggers](https://csstriggers.com/)

## When to use animation

Don't let animation cause performance problems--it shouldn't stop users from using your site. 
- Navigational cues that apply movement to elements that indicate the next action a user should take, like scrolling or clicking the next button.
  - can familiarize your users with where to find certain actions and how to do things
- Also good for feedback when they interact with UI elements
  - ID the user touchpoints and add appropriate animation:
    - hover buttons
    - form inputs
    - text input's line color - like an incorrect password or pin
- Navigation and page transitions
  - Expand selections into a new page that blocks out the page you made the selection from
  - Gives the user the feeling they are going deeper into your app
  - These are common on mobile, but not on desktop. This is all possible with React and Vue.js
- Indicate the status of something in progress
  - let users know what is happening with your site! Its busy, not frozen
  - ex: loading animation during a download or form submission, spinning wheel, progress bar

## Planning animations

This is called _storyboarding_. You visually plan out the sequence of events in a low-fidely way, from beginning, to middle, to end:
- to storyboard a button, draw three blank squares that represent resting state, hover state, and final state
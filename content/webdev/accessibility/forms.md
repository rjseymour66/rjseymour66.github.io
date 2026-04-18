---
title: "Forms"
# linkTitle: ""
weight: 100
# description:
---

Forms are often the most critical path in a product, covering login, checkout, search, and account management. An inaccessible form blocks a user from completing the task entirely. Follow these basic principles:

- Apply native form elements when possible
- Apply form elements for their intended purpose
- Keep forms short
- Label and describe all fields
- Inform users about changes to the form

People do not enjoy filling in forms, so keep them short. Before you add a question to a form, ask yourself the following questions:

- Do you need the information to deliver the service?
- Why do you need the information?
- What will you do with the information?
- Which users need to give you the information?
- How will you check that the information is accurate?
- How will you keep the information up-to-date and secure?

## Labels

Always associate a label with its form element with the `for` and `id` attributes. This association means screen readers announce the label text when the user focuses the input. Without it, a screen reader user on a sign-up form may hear "edit text" with no indication of what to type.

The following example shows the standard label-input association:

```html
<label for="email">Email address</label>
<input type="email" name="email" id="email" />
```

Sometimes you cannot add a visible label. In that case, create a reference to an existing element with `aria-labelledby`. Here, `aria-labelledby` associates the adjacent button with the input field, so the screen reader announces "Search" when the user focuses the input:

```html
<form action="">
    <input type="text" name="" id="" aria-labelledby="btn_search" />
    <button
        id="btn_search">
        Search
    </button>
</form>
```

### Positioning

Place the label directly above the input. Research consistently shows this layout reduces completion time and errors compared to side-by-side or below placement.

### Placeholders

Be careful with placeholders. Placeholders are not substitutes for labels for these reasons:

- Placeholder text disappears when the user starts typing
- The browser may prepopulate fields, which means users have to remove content to verify what the field expects
- Color styles usually do not provide sufficient contrast
- Longer text may get cut off
- Users may mistake placeholder text for a pre-filled value

## Descriptions

Add information about the form in the following locations:

- Intro: A sentence or two before the first field explaining what the form does and what information to gather before starting.
- In the label element, either before or beside the input. For longer instructions, apply `aria-describedby` to connect descriptions to the input element.

The following example connects a hint to a password field. When the user focuses the input, the screen reader announces the label and then the hint:

```html
<label for="password">Password</label>
<input type="password" id="password" aria-describedby="password-hint" />
<p id="password-hint">Must be at least 8 characters and include a number.</p>
```

### autocomplete

Apply the `autocomplete` attribute so the browser can fill in known information. This especially benefits users with motor disabilities who find typing difficult. The following example applies the `bday` value:

```html
<label for="bday">Birthday (dd.mm.yyyy)</label>
<input type="text" autocomplete="bday" id="bday" />
```

The HTML Standard has a [full list of autocomplete attributes](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofilling-form-controls:-the-autocomplete-attribute).


## Errors

Read [A Guide to Accessible Form Validation](https://www.smashingmagazine.com/2023/02/guide-accessible-form-validation/).

When a user enters invalid data, place error messages close to the input field. A screen reader user needs to hear the error immediately after the label, not find it somewhere else on the page. Apply these attributes:

- `aria-invalid="true"` to mark a field as invalid
- `aria-describedby="<hint-or-error-msg-ID>"` to associate the error message with the field. This attribute accepts multiple space-delimited values, so you can reference both a hint and an error at once.

The following example shows an inline validation pattern. The error message is associated with the input so the screen reader announces it when the user focuses the field:

```html
<label for="email">Email address</label>
<input
    type="email"
    id="email"
    aria-invalid="true"
    aria-describedby="email-error"
/>
<p id="email-error">Enter a valid email address, for example, name@example.com.</p>
```

If you do server-side validation, consider rendering a summary above the form that lists all errors. Make the summary focusable and link each error to its field so keyboard users can jump directly to the problem:

```html
<div role="region" aria-labelledby="error_heading" tabindex="0">
    <h2 id="error_heading">2 issues found</h2>
    <ul>
        <li><a href="#email">Enter a valid email address</a></li>
        <li><a href="#password">Enter a valid password</a></li>
    </ul>
</div>
```

### Postel's law

"Be conservative in what you do, be liberal in what you accept from others."

Applied to forms, this means writing backend code that sanitizes and normalizes user input so users do not need to be precise. For example, accept phone numbers with or without hyphens and spaces, then normalize them in the backend. This reduces friction and error rates without compromising data quality.

## Grouping fields

Group related form questions in a `<fieldset>` element and always provide a `<legend>` as its first child. The legend gives the group a shared context that individual labels cannot provide on their own. For example, a payment method radio group needs the legend "Payment method" so screen reader users understand that "Credit card", "PayPal", and "Bank transfer" are options within the same choice, not independent questions:

```html
<fieldset>
    <legend>Payment method</legend>
    <label><input type="radio" name="payment" value="card"> Credit card</label>
    <label><input type="radio" name="payment" value="paypal"> PayPal</label>
    <label><input type="radio" name="payment" value="bank"> Bank transfer</label>
</fieldset>
```

## Long forms

If a form collects a lot of information, break it into multiple steps where each step is completed on its own page. Follow these guidelines:

- Explain the process and list any information the user should gather on a page before the first step.
- Show a progress indicator so users know how many steps remain.
- Update the page `<title>` on each step to reflect the current step, for example, "Checkout (step 2 of 3)".

## Links and references

- [web.dev Lean Accessibility](https://web.dev/learn/accessibility/forms/)
- [Creating Accessible Forms](https://webaim.org/techniques/forms/advanced)
- [Designing good questions](https://www.gov.uk/service-manual/design/designing-good-questions)
- [Autocomplete](https://saptaks.blog/posts/there-is-a-lot-more-to-autocomplete-than-you-think.html)
- [Create accessible forms](https://www.a11yproject.com/posts/how-to-write-accessible-forms/)
- [Using fieldset and legend elements](https://accessibility.blog.gov.uk/2016/07/22/using-the-fieldset-and-legend-elements/)
- [Forms tutorial: Grouping controls](https://www.w3.org/WAI/tutorials/forms/grouping/)

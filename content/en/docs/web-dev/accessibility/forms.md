---
title: "Forms"
# linkTitle: ""
weight: 100
# description:
---

Basic form principles:
- Use native form elements when possible
- Use form elements for their intended purpose
- Keep forms short
- Label and describe all fields
- Inform users about changes to the form


People don't like forms, so keep them short. Before you add a question to a form, ask yourself the following questions:
- Doyou need the information to deliver the service?
- Whey do you need the information?
- What will you do with the information?
- Which users need to give you the information?
- How will you check that the information is accurate?
- How will you keep the information up-to-date and secure?

## Labels

Always associate a label with its form element with the `for` and `id` attributes. Sometimes, you can't do that, so create a reference to an existing element with `aria-labelledby`. Here, `aria-labelledby` associates the adjacent button with the input field:

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

Place the label directly above the input. There have been many studies that show this is the best way to structure the form.

### Placeholders

Be careful with placeholders. Placeholders are not substitutes for labels for these reasons:
- Placeholder text disappears when you type
- Browser might prepopulate fields, which means that users have to remove content to verify the field
- Color styles usually don't provide enough contrast
- Longer text might get cut off
- Might be mistaken for a value

## Descriptions

Add information about the form in the following locations:
- Intro
- In the label element, either before or beside the input. For longer instructions, use `aria-describedby` to connect descriptions to the input element.

### autocomplete

Use this attribute so the browser can fill in the information, if possible:

```html
<label for="bday">Birthday (dd.mm.yyyy)</label>
<input type="text" autocomplete="bday" id="bday" />
```

Here, we use the `bday` value. The HTML Standard has a [full list of autocomplete attributes](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofilling-form-controls:-the-autocomplete-attribute).


## Errors

Read [A Guide to Accessible Form Validation](https://www.smashingmagazine.com/2023/02/guide-accessible-form-validation/)

When a user enters invalid data into an input, place error messages close to the input field with these attributes:
- `aria-invalid="true"` to mark a field as invalid
- `aria-describedby="<hint-or-error-msg-ID>"` to associate with the hint or error message. Accepts multiple, space-delimited values.

If you do server-side validation, consider rendering a section above the form that lists errors. This example is focusable and references the 

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

"Be conservative in what you do, be liberal in what you accept from others"

Applied to forms, this means that you can write some backend code that sanitizes form inputs so users don't have to be so precise. This improves the user experience.

## Grouping fields

Group similar form questions and their related elements in a `<fieldset>` element. Always provide a `<legend>` as the first element in the fieldset.

## Long forms

If a form collects a lot of information, break it up into multiple steps, where each step is completed on a page:
- Explain the process and any info that the user should collect on a page before the first steps

## Links and references

- [web.dev Lean Accessibility](https://web.dev/learn/accessibility/forms/)
- [Creating Accessible Forms](https://webaim.org/techniques/forms/advanced)
- [Designing good questions](https://www.gov.uk/service-manual/design/designing-good-questions)
- [Autocomplete](https://saptaks.blog/posts/there-is-a-lot-more-to-autocomplete-than-you-think.html)
- [Create accessible forms](https://www.a11yproject.com/posts/how-to-write-accessible-forms/)
- [Using fieldset and legend elements](https://accessibility.blog.gov.uk/2016/07/22/using-the-fieldset-and-legend-elements/)
- [Forms tutorial: Grouping controls](https://www.w3.org/WAI/tutorials/forms/grouping/)
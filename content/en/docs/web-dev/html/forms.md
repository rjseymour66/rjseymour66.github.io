---
title: "Forms"
# linkTitle: ""
weight: 110
# description:
---

[MDN docs: 22 types of inputs](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types)

The `<form>` element is a document landmark that contains interactive controls for submitting information. HTML `<forms>` can do the following without help from JS:

- require the user to select form controls or enter a value
- define specific criteria that the value must match
- perform client-side constraint validation and prevent submission until it passes the criteria

To turn off client-side validation and let the user save progress until ready to submit, set the `novalidate` attribute on the form, or the `formnovalidate` attribute on a button.

## Submitting forms

A form is submitted when you select the button in the form. You can create a button with an `<input>` or `<button>` element.

```html
<!-- value is the button text -->
<input type="submit" value="Submit Form" />
<button type="submit">Submit Form</button>
```

Form attributes define the HTTP method used to submit the form and the URL to submit it to:

- `action`: URL that processes the form. By default, form data is sent to the current page if you don't set `action`
- `method`: HTTP method used for the form

The data sent in the request is identified by the form controls' `name` attribute. If the form control is not nested in the form, you can link it with the `id` attribute on the form control.

- GET requests send the form data as a string of `name=value` pairs appended to the `action` URL
- POST requests append the data to the body of the HTTP request. Always use POST when sending secure data

## After submitting the form

Forms submit data of all info that includes name and value attributes, except non-selected checkboxes and radio buttons. For example, in a GET request:

- `name` attribute is the key in the URL string
- `value` is the value in the URL string. If no value, then it submits the inner text.

## Radio buttons

You can only select one radio button at a time. Each button has the same `name` attr:

- The `name` and the `value` are submitted with the form
- you can include as many radio groups as you want on a page, just use a unique `name` for each
- if you reuse `name` for more than one group, the first radio group `name` is submitted

```html
<fieldset>
  <legend>Who is your favorite student?</legend>
  <ul>
    <li>
      <label>
        <input type="radio" value="blendan" name="machine" /> Blendan Smooth
      </label>
    </li>
    <li>
      <label>
        <input type="radio" value="hoover" name="machine" /> Hoover Sukhdeep
      </label>
    </li>
    <li>
      <label>
        <input type="radio" value="toasty" name="machine" /> Toasty McToastface
      </label>
    </li>
  </ul>
</fieldset>
```

To have a default selection, use the `checked` attribute. The checked button matches these CSS selectors:

```css
.radio:default
.radio:checked
```

Add the `required` attribute to make a selection for a radio group requried.

## Checkboxes

Checkboxes are similar to radio buttons, but you can submit multiple checkboxes in a group. Other properties:

- If you don't include a `value` for the checkbox, it is submitted as `on`
- you can set multiple checkboxes as `required`
- they can all have the same `name` attr, but that might make it difficult to distinguish when submitting the form

## Labels and fieldsets

- Every form control must have a label.
- groups of form controls are labeled by the contents of the `<legend>` of the `<fieldset>` that groups them

Labels provide form controls accessible names. You need to include one. Labels and inputs are linked by their `for` and `id` attrs:

- `for` attribute on label
- `id` attribute on input

```html
<label for="full_name">Your name</label>
<input type="text" id="full_name" name="name" />
```

Implicit labels nest the `<input>` within the opening and closing `<label>` tags:

```html
<label
  >Your name
  <input type="text" name="name" />
</label>
```

To label a group of inputs, group the elements in a `<fieldset>` with the `<legend>` being the label for the group:

```html
<fieldset>
  <legend>Who is your favorite student?</legend>
  <ul>
    <li>
      <label>
        <input type="radio" value="blendan" name="machine" /> Blendan Smooth
      </label>
    </li>
    <li>
      <label>
        <input type="radio" value="hoover" name="machine" /> Hoover Sukhdeep
      </label>
    </li>
    <li>
      <label>
        <input type="radio" value="toasty" name="machine" /> Toasty McToastface
      </label>
    </li>
  </ul>
</fieldset>
```

## Built-in validation

There are CSS selectors that match attributes added to form controls:

- `:required`: if `required` is set
- `:optional`: if `required` is set
- `:default`: if `checked` is hard-coded
- `:enabled`: if the element is interactive and whether or not `disabled` is set
- `:disabled`: if the element is interactive and whether or not `disabled` is set
- `:read-write`: elements with `contenteditable` set and form controls that are editable by default
- `:read-only`: normally writable elements that have `readonly` set

To style form controls as the user enters information, use these selectors. They will toggle on and off, depending on the state of the input:

- `:valid`
- `:invalid`
- `:in-range`
- `:out-of-range`

Applied CSS is updated continuously based on the UI state. For example, if you are entering an email, and the current value is non-null but does not match the acceptable constraints, then the `:invalid` CSS pseudo-class will match it.

The browser has built-in error messages for fields, but they are displayed when the form is submitted.
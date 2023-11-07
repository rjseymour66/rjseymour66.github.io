---
title: "Forms"
weight: 50
description: >
  Creating and styling forms.
---

## Links

- [Basic native form controls](https://developer.mozilla.org/en-US/docs/Learn/Forms/Basic_native_form_controls)
- [The HTML5 input types](https://developer.mozilla.org/en-US/docs/Learn/Forms/HTML5_input_types)
- [Other form controls](https://developer.mozilla.org/en-US/docs/Learn/Forms/Other_form_controls)

## Form element

The form is a container (like a div) that wraps all the inputs that a user interacts with. Each form **must** have the following attributes:
- `action`: accepts the URL value that the form sends its data to
- `method`: tells the browswer which HTTP method the form uses.
  
  You will use the following methods the most:
  - `GET` when you want to retrieve something from a server.
  - `POST` when you want to change something on a server.

A form usually submits data to a server, but you can use form controls outside of a form element to get information from the user.

## Form controls

A form control is an element that the user interacts with, such as a text box, dropdown, or checkbox.

### inputs and labels

The input element accepts text, and the label element tells users what information the input element expects.

```html
<label for="user_email">Email Address:</label>
<input type="email" id="user_email" name="email" placeholder="you@example.com">
```

> The `input` element is a _void element_---it does not require a closing tag. For example: `<input type="email" />`.

You can add the following attributes to an `<input>` element in the `attribute="value"` format:

| Attribute | Association | Description |
|:----------|:------------|:------------|
| `type`    |   | Tells the browser what type of data to expect. Helps to validate the user entry. Examples inlcude `text`, `email`, `password`, `number`, `date`, `radio`, `checkbox`, `tel`, `hidden`, `search`, `range` (slider)  |
| `id`      | \<label\\> `for` attribute | Associates a label to an input element for assistive technology--focusses on the input when the label is clicked. |
| `placeholder` |   | Guide users on what to add to input fields and how to format it.  |
| `name`  | The key name for this value in the request object sent to the server.  | Required, or the server ignores the data. Tells the backend know what this data represents.  |

This [Sitepoint article](https://www.sitepoint.com/html-forms-constraint-validation-complete-guide/) lists additional options.

> To associate a label with an input, the label's `for` and the input's `id` attributes must have the exact same value.
> 
> You can also nest the input within the label:
> 
> ```html
> <label for="name">
> Name: <input type="text" id="name" name="user_name" />
> </label>
> ```
> The `for`/`id` pattern is considered a best practice because of assistive technologies.

Use `type="hidden"` to hide an input from the user so you can do things like send a timestamp for when the form was submitted. The `name` and `value` attributes are required:

```html
<input type="hidden" id="timestamp" name="timestamp" value="1286705410" />
```

> There is also an `<output>` element that allows you to display the calculation of an element such as a slider. You can associate it with another element with the `name` attribute.

### textarea

You can set the size, and users can click and drag to make it bigger or smaller. You can optionally add some initial content between the opening and closing tags:

```html
<textarea rows="20" cols="30">Tell me something...</textarea>
```

Accepts these three attributes:
- `cols`: default is 20
- `rows`: default is 2
- `wrap`: Accepts these settings:
  - `soft`: submitted text is not wrapped but rendered text is 
  - `hard`: submitted and rendered text is wrapped. Must have `cols` setting. 
  - `off`: no wrapping.

`resize` CSS property lets you change the resize behavior. [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/resize).

### Select dropdown

Select dropdowns contain options that you can select. The `value` is what is sent to the server when the form is submitted.

The `selected` attribute specifies the value that is selected by default:

```html
<select name="Car">
  <option value="mercedes">Mercedes</option>
  <option value="tesla">Tesla</option>
  <option value="volvo" selected>Volvo</option>
</select>
```

You can also divide the options with `optgroup`, which accepts a `lable="Name"` attribute to name the optgroup:

```html
<select name="fashion">
  <optgroup label="Clothing">
    <option value="t_shirt">T-Shirts</option>
    <option value="sweater">Sweaters</option>
  </optgroup>
  <optgroup label="Foot Wear">
    <option value="sneakers">Sneakers</option>
    <option value="boots">Boots</option>
  </optgroup>
</select>
```

### Autocomplete box

You can provide suggestions for a dropdown input with the `<datalist>` element that contains a list of `<option>` elements. 

Bind an `<input>` element to the list with the `input:list` and `datalist:id` attributes. For example:

```html
<label for="myFruit">What's your favorite fruit?</label>
<input type="text" name="myFruit" id="myFruit" list="mySuggestion" />
<datalist id="mySuggestion">
  <option>Apple</option>
  <option>Banana</option>
  <option>Blackberry</option>
</datalist>
```

### Checkable items

Checkable items---radio buttons and checkboxes---send their value to the server only if they are checked (unlike other input elements that send information if they have a `name` attribute). If they are not checked, nothing is sent.

If they are checked but have no `value` attribute, the name of the checkable item is sent with the value `on`.

#### Radio buttons

Use radio buttons when you have 5 or fewer options to choose from so the user can see all options at once instead of hiding them behind a dropdown. 

> If you have a set of radio buttons, nest them in a [\<fieldset\> element](#organization).

The `name` attribute associates radio buttons with one another. When more than one radio button has the same `name` attribute, you can select only one of them at a time.

Set the default value with the `checked` attribute. Notice that the `<lable>` is after the `<input type="radio">`:

```html
<div>
  <input type="radio" id="child" name="ticket_type" value="child">
  <label for="child">Child</label>
</div>

<div>
  <input type="radio" id="adult" name="ticket_type" value="adult" checked>
  <label for="adult">Adult</label>
</div>
```

#### Checkboxes

> Styling checkboxes is a PITA. [This article](https://moderncss.dev/pure-css-custom-checkbox-style/) provides some tips.

Checkboxes are like radio buttons, but you can select more than one at a time:

```html
<h1>Pizza Toppings</h1>

<div>
  <input type="checkbox" id="sausage" name="topping" value="sausage">
  <label for="sausage">Sausage</label>
</div>

<div>
  <input type="checkbox" id="onions" name="topping" value="onions">
  <label for="onions">Onions</label>
</div>
```
The `name` attribute associates checkboxes with one another.

You can have a single checkbox if you are asking the user to do something like subscribe to a mailing list. Use the `checked` attribute to check it by default:

```html
<div>
  <input type="checkbox" id="newsletter" name="news_letter" checked>
  <label for="newsletter">Send me the news letter</label>
</div>
```

## Buttons

Users click buttons to submit forms or trigger other actions. The button attribute `type` tells the browser what the button does:

| Type   | Description |
|:-------|:------------|
| `submit` | Default value. Submits the form that the button is contained in. |
| `reset`  | **Rarely used**. Clears all data that a user entered into a form and sets the forms back to their default values. |
| `button` | Generic button that you can use for anything. Commonly used with JS to create interactive UIs. |


> By default, a form button `type` attribute is set to `submit`. Always set the `type` attribute so that the button does not submit the form by accident.

You should try to place the `<button>` attribute within the `<form>` element. If you cannot, you can link the form and button with `id` attributes:

```html
<section class="form">
    <form action="" method="post" id="odin-form"></form>
</section>
<section class="cta">
    <button type="submit" id="odin-form"></button>
</section>
```

## Organization

You can group form inputs withe the `<fieldset>` element, and label each `<fieldset>` section with the `<legend>` element:

```html
<fieldset>
  <legend>Contact Details</legend>

  <label for="name">Name:</label>
  <input type="text" id="name" name="name">

  <label for="phone_number">Phone Number:</label>
  <input type="tel" id="phone_number" name="phone_number">

  <label for="email">Email:</label>
  <input type="email" id="email" name="email">
</fieldset>
```

## UX and styles

Here are some helful articles:
- [UX And HTML5: Let’s Help Users Fill In Your Mobile Form](https://www.smashingmagazine.com/2018/08/ux-html5-mobile-form-part-1/)
- [7 Basic Best Practices for Buttons](https://www.uxmatters.com/mt/archives/2012/05/7-basic-best-practices-for-buttons.php)

You can wrap labels and inputs in the following elements:
- `<li>` (In either `<ul>` or `<ol>`'s, good for checkboxes or radio buttons)
- `<p>`
- `<div>`

You can also wrap larger form portions in a `<section>` element.



## Dropdowns

### Autocomplete box

You can provide suggestions for a dropdown input with the `<datalist>` element that contains a list of `<option>` elements. 

Bind an `<input>` element to the list with the `input:list` and `datalist:id` attributes. For example:

```html
<label for="myFruit">What's your favorite fruit?</label>
<input type="text" name="myFruit" id="myFruit" list="mySuggestion" />
<datalist id="mySuggestion">
  <option>Apple</option>
  <option>Banana</option>
  <option>Blackberry</option>
</datalist>
```

## Meter bar

The meter bar is complicated. There are low, high, and optimum attributes. [Read the docs](https://developer.mozilla.org/en-US/docs/Learn/Forms/Other_form_controls#meters_and_progress_bars), but here is an example:

```html
<meter min="0" max="100" value="75" low="33" high="66" optimum="0">75</meter>
```
## Progress bars

Progress bars represent a value that changes over time up to a maximum value, like the total number of files downloaded or progress in a questionnaire:

```html
<progress max="100" value="75">75/100</progress>
```

## Styling forms

Most form widgets are easy to style except the following:

- Checkboxes
- Radio buttons
- Search bars `<input type="search">`

Other items cannot be styled with only CSS:

- Color picker `<input type="color">`
- Date controls such as `<input type="datetime-local">`
- Slider selectors `<input type="range">`
- File selectors `<input type="file">`
- Dropdown elements (`<select>`, `<option>`, `<optgroup>` and `<datalist>`)
- `<progress>` and `<meter>`

### Form reset

Default browser styles around fonts are inconsistent, and each form element has its own default rules for `border`, `padding`, and `margin`. Add the following to make the form font consistent with the rest of your content:

```css
button,
input,
select,
textarea {
  width: 150px;
  padding: 0;
  margin: 0;
  box-sizing: border-box;

  font-family: inherit;
  font-size: 100%;

  /* removes system-level styling */
  appearance: none;
}
```

### Legend placement

If you want to move the legend description text, you have to make the fieldset `position: relative;` and the legend `position: absolute;`:

```css
fieldset {
  position: relative;
}

legend {
  position: absolute;
  bottom: 0;
  right: 0;
}
```

### textarea

These default to `display: inline-block;`. The important attributes are `resize` and `overflow`:

- In general, do not restrict users with `resize`
- `overflow` makes the element render consistently across browsers. Setting to `auto` usually fixes this.

```css
textarea {
  display: block;

  padding: 10px;
  margin: 10px 0 0 -10px;
  width: 100%;
  height: 90%;

  border-right: 1px solid;

  /* resize  : none; */
  overflow: auto;
}
```

### Search input

Safari restricts some styling (`height`, `font-size`), so you need to add `appearance: none;` to search inputs:

```css
input[type="search"] {
  appearance: none;
}
```

When a search field is not empty, there is an 'x' in the right of the box when it is in focus. When it is not in focus, it disappears in Edge and Chrome, but not Safari. You can make this behavior consistent with the following:

```css
input[type="search"]:not(:focus, :active)::-webkit-search-cancel-button {
  display: none;
}
```

## radio and checkboxes

These elements are difficult to style bc of the default appearance. So, you should first remove that, then optionally style the checkbox appearance.

[Pure CSS Custom Checkbox Style](https://moderncss.dev/pure-css-custom-checkbox-style/)

The following example creates a circular checkbox, where the checked value is slightly smaller than the checkbox. The commented out portions use a checkmark as the content.

First, the HTML:

```html
<form>
  <div class="form-row checkbox">
      <input type="checkbox" name="read" id="read" checked>
      <label for="read">Read?</label>
  </div>
</form>
```

The CSS:
```css
.form-row.checkbox {
  flex-direction: row;
  gap: 0.5rem;
}

input[type="checkbox"] {
  appearance: none;
}

input[type="checkbox"] {
  position: relative;
  width: 1.25rem;
  height: 1.25rem;
  align-self: center;

  border: 2px solid var(--clr-form-border);
  border-radius: 50%;

  color: var(--clr-primary);
}

input[type="checkbox"]::before {
  /* content: "✔"; */

  content: " ";
  background-color: var(--clr-form-border);
  height: 0.9rem;
  width: 0.9rem;
  /* color: blue; */
  position: absolute;
  font-size: 1.5rem;

  border-radius: 50%;

  right: 1px;
  top: 1px;

  /* right: -5px;
  top: -10px; */

  visibility: hidden;
}

input[type="checkbox"]:checked::before {
  /* Use `visibility` instead of `display` to avoid recalculating layout */
  visibility: visible;
}
```

For example, the following styles add a checkmark in the checkbox and apply other general styles:

```css
input[type="checkbox"] {
  appearance: none;
}

input[type="checkbox"] {
  position: relative;
  width: 1em;
  height: 1em;
  border: 1px solid gray;
  /* Adjusts the position of the checkboxes on the text baseline */
  vertical-align: -2px;
  /* Set here so that Windows' High-Contrast Mode can override */
  color: green;
}

input[type="checkbox"]::before {
  content: "✔";
  position: absolute;
  font-size: 1.2em;
  right: -1px;
  top: -0.3em;
  visibility: hidden;
}

input[type="checkbox"]:checked::before {
  /* Use `visibility` instead of `display` to avoid recalculating layout */
  visibility: visible;
}

input[type="checkbox"]:disabled {
  border-color: black;
  background: #ddd;
  color: gray;
}
```

## Difficult items

These include the following:

- Color picker `<input type="color">`
- Date controls such as `<input type="datetime-local">`. You can style the containing box, but you can't style any of the popups.
- Slider selectors `<input type="range">`
- File selectors `<input type="file">`
- Dropdown elements (`<select>`, `<option>`, `<optgroup>` and `<datalist>`)
- `<progress>` and `<meter>`

You can "reset" (or get close to resetting them) with the following styles:

```css
button,
label,
input,
select,
progress,
meter {
  display: block;
  font-family: inherit;
  font-size: 100%;
  margin: 0;
  box-sizing: border-box;
  width: 100%;
  padding: 5px;
  height: 30px;
}

/* box shado inside */
input[type="text"],
input[type="datetime-local"],
input[type="color"],
select {
  box-shadow: inset 1px 1px 3px #ccc;
  border-radius: 5px;
}
```

## Select and datalist

The down arrow that indicates it is a dropdown is inconsistent across browsers, and sizing is an issue. Start by adding `appearance`:

```css
select {
  appearance: none;
}
```

To add an arrow, you have to add a wrapper to the `<select>` element because `::before` and `::after` do not work on `<select>` elements:

```html
<label for="select">Select a fruit</label>
<div class="select-wrapper">
  <select id="select" name="select">
    <option>Banana</option>
    <option>Cherry</option>
    <option>Lemon</option>
  </select>
</div>
```

The wrapper is the parent context for the arrow positioning:

```css
.select-wrapper {
  position: relative;
}

.select-wrapper::after {
  content: "▼";
  font-size: 1rem;
  top: 6px;
  right: 10px;
  position: absolute;
}
```

Styling the dropdown box requires a custom library--you can only inherit font with the CSS.

## Sliders (range)

You can't do much with this input type other than remove it completely and replace it with a color of your choice. For example, the following creates a red slider:

```css
input[type="range"] {
  appearance: none;
  background: red;
  height: 2px;
  padding: 0;
  outline: 1px solid transparent;
}
```

## Color pickers

You can only remove the border:

```css
input[type="color"] {
  border: 0;
  padding: 0;
}
```

## File uploader

You cannot style the default button at all, but you can style it so the button does not display, and then add your own button:

```html
<form>
  <div>
    <label for="file">Choose a file to upload</label>
    <input id="file" name="file" type="file" multiple />
    <ul id="file-list"></ul>
  </div>
  <div><button>Submit?</button></div>
</form>
```

```css
label[for="file"] {
  box-shadow: 1px 1px 3px #ccc;
  background: linear-gradient(to bottom, #eee, #ccc);
  border: 1px solid rgb(169, 169, 169);
  border-radius: 5px;
  text-align: center;
  line-height: 1.5;
}

label[for="file"]:hover {
  background: linear-gradient(to bottom, #fff, #ddd);
}

label[for="file"]:active {
  box-shadow: inset 1px 1px 3px #ccc;
}
```

Listing the files requires JS. Look in the script tag in the [MDN repo](https://github.com/mdn/learning-area/blob/main/html/forms/styling-examples/styled-file-picker.html).

## Meter and progress bars

These are impossible to style. For progress bars, use [ProgressBar.js](https://kimmobrunfeldt.github.io/progressbar.js/#examples).

## Pseudo-classes

A pseudo-class targets an existing element. Some of the most common ones include:

| Pseudo-class                                             | Description                                                                                                                                                                                                                                                                                                      |
| :------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `:required`, `:optional`                                 | Target elements that can be required                                                                                                                                                                                                                                                                             |
| `:valid` and `:invalid`, `:in-range` and `:out-of-range` | Target form controls that are valid/invalid according to form validation constraints set on them, or in-range/out-of-range.                                                                                                                                                                                      |
| :enabled, :disabled, and :read-only, :read-write         | Target elements that can be disabled (e.g. elements that support the disabled HTML attribute), based on whether they are currently enabled or disabled, and read-write or read-only form controls (e.g. elements with the readonly HTML attribute set).                                                          |
| `:checked`, `:indeterminate`, and `:default`             | Respectively target checkboxes and radio buttons that are checked, in an indeterminate state (neither checked or not checked), and the default selected option when the page loads (e.g. an `<input type="checkbox">` with the checked attribute set, or an `<option>` element with the selected attribute set). |

[Table is from MDN](https://developer.mozilla.org/en-US/docs/Learn/Forms/UI_pseudo-classes#what_pseudo-classes_do_we_have_available).

### Required

If an element has a `required` or `optional` attribute, you can style them like so:

```css
input:required {
  border: 1px solid black;
}

input:optional {
  border: 1px solid silver;
}
```

### Generating content with spans

Form inputs don't support generated content, because generated content is placed relative to an element's formatting box and inputs don't have a formatting box.

_The trick is to use a `<span>` to attach generated content_.

Place the span after the input element:

```html
<div>
  <label for="fname">First name: </label>
  <input id="fname" name="fname" type="text" required />
  <span></span>
</div>
```

> If the label and input take up 100% of the container width, the span will be on the next line. To fix this, make the `<div>` a flex container with `flex-flow: row wrap;` so that the `<label>` and `<input>` still sit on their own line, but the `<span>` is right after the `<input>` because it has a width of 0.

Now, you can target the span and add `position: relative` so that you can `position: absolute` a `::before` or `::after` pseudo element.

```css
/* targets the span */
input + span {
  position: relative;
}

input:required + span::after {
  font-size: 0.7rem;
  position: absolute;
  content: "required";
  color: white;
  background-color: black;
  padding: 5px 10px;
  top: -26px;
  left: -70px;
}
```

### Valid and invalid data

If a form has constraint limitations, you can style them based on whether they meet those constraints. Also, consider the following:

- No constraints means the element is always valid and you can target it with `:valid`.
- Elements with `required` and no value are invalid, and you can target them with `:invalid` and `:required`.
- Elements with built-in validation like `url` or `email` types are invalid when the values don't meet those constraints.
- Elements with values outside a `min` or `max` range are `:invalid` or `:out-of-range`.

Use the [span](#generating-content-with-spans) trick to add pseudo-elements based on element state. The following adds a red 'x' or green check after the input, based on the value:

```css
input + span {
  position: relative;
}

input + span::before {
  position: absolute;
  right: -20px;
  top: 5px;
}

input:invalid {
  border: 2px solid red;
}

input:invalid + span::before {
  content: "✖";
  color: red;
}

input:valid + span::before {
  content: "✓";
  color: green;
}
```

### In-range and out-of-range

Use these pseudo-classes when the inputs are outside of a range defined by `min` or `max` attributes.

> You could use `:valid` or `:invalid`, but `:in-range` and `:out-of-range` are more semantically correct.


### Enabled and disabled input

You can gray-out elements if users don't need to fill them out. A common example is shipping and billing info---if the addresses match, then you don't have to fill out the billing.

[This example](https://developer.mozilla.org/en-US/docs/Learn/Forms/UI_pseudo-classes#styling_enabled_and_disabled_inputs_and_read-only_and_read-write) shows how to do that, including the JS.

### Read-only and read-write

You might need to display form data that the user cannot edit--like a confirmation page before the final form submission.

You can do this with the `readonly` attribute on the input element, and the `:read-only` pseudo-class. To allow editing, use the `read-write` attribute (this is the input element's default state).

The MDN docs have a [full example here](https://developer.mozilla.org/en-US/docs/Learn/Forms/UI_pseudo-classes#read-only_and_read-write).

### Radio and checkbox states

`:checked` is useful when you reset the checkbox styling with `appearance: none;` and you need to add your own styling.

`:default` matches checkboxes that are checked by default (with the `default` attribute.)

### More pseudo-classes

Check out MDN for more [less-used pseudo-classes](https://developer.mozilla.org/en-US/docs/Learn/Forms/UI_pseudo-classes#more_pseudo-classes).




## Form element

When you leave the `action` attribute blank, the form submits to the same URL, which lets us test the form.

You can style a `<form>` element as you would style a `<div>`

## Styling

### Namespacing styles

Avoid creating global `input[type="text"]` styles, and try to namespace the styles:

```html
<div class="form-row">
  <label for="full-name">Name</label>
  <input id="full-name" name="full-name" type="text" />
</div>
```

```css
.form-row input[type="text"] {
  /* styles... */
}

.form-row label {
  /* styles... */
}
```

### Mobile vs desktop

Try styling mobile forms with `flex-direction: column;` and desktop with `flex-direction: row;`. If you don't use flex, still structure them as columns or rows.

### Radio buttons

Apply the following to every radio button group:

- Wrap in a <fieldset> and label it with <legend>
- Associate a <label> element with each radio button
- Use the same name attribute for each radio button in the group.
- Use different value attributes for each radio button.

Radio buttons do the following:

- send their `value` to the server
- must have a `name` attribute to associate them with the radio button group.

### Select elements

The `<select>` element is hard to style because the default styles vary so widely across browsers and devices. There is not much you can do, so don't overdo it.

To style the text, you have to set `appearance: none;` or it won't work correctly on Chrome or Safari.

### Checkboxes

You can select any number of checkboxes--they are not part of a group. This means that you do not need to wrap them in a `<fieldset>` element.

### Submit button

If the form's `action` attribute is blank, you can submit the form and see the query parameters in the browser URL box.



## Form validation

Validate data so it is in the correct format, and that it is secure for both the users and the application.

- This [Sitepoint](https://www.sitepoint.com/html-forms-constraint-validation-complete-guide/) article provides a summary of attributes, including autocomplete for SMS OTP.
- [web.dev SMS OTP](https://web.dev/articles/sms-otp-form#autocomplete%22one-time-code%22)
- [Good codepen example](https://www.silocreativo.com/en/css-rescue-improving-ux-forms/)

### Attributes

| Attribute   | Type      | Description |
| :---------- | :-------- | :-----------|
| `required`  | Boolean   | Makes any input field required. YOu should also add an asterisk to a required field label.                                                               |
| `minlength` | key/value | Minimum number of text characters. Can be combined with `maxlength`.                                                                                     |
| `maxlength` | key/value | Maximum number of text characters. Can be combined with `minlength`.                                                                                     |
| `min`       | key/value | Minimum number value accepted. Can be combined with `max`.                                                                                               |
| `max`       | key/value | Maximum number value accepted. Can be combined with `min`.                                                                                               |
| `pattern`   | key/value | `<input>` elements only. Must match the regular expression. Common use cases include zipcodes or CC numbers. Use with `placeholder` to provide guidance. |


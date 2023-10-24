---
title: "Forms"
weight: 50
description: >
  Creating and styling forms.
---


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

> `input` elements do not have closing tags. For example: `<input type="email" />`.

You can add the following attributes to an `<input>` element in the `attribute="value"` format:

| Attribute | Association | Description |
|:----------|:------------|:------------|
| `type`    |   | Tells the browser what type of data to expect. Helps to validate the user entry. Examples inlcude `text`, `email`, `password`, `number`, `date`, `radio`, `checkbox`  |
| `id`      | label `for` | Associates a label to an input element for assistive technology--focusses on the input when the label is clicked. |
| `placeholder` |   | Guide users on what to add to input fields and how to format it.  |
| `name`  | The key name for this value in the request object sent to the server.  | Required, or the server ignores the data. Tells the backend know what this data represents.  |

### textarea

You can set the size, and users can click and drag to make it bigger or smaller. You can optionally add some initial content between the opening and closing tags:

```html
<textarea rows="20" cols="30">Tell me something...</textarea>
```

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

### Radio buttons

Use radio buttons when you have 5 or fewer options to choose from so the user can see all options at once instead of hiding them behind a dropdown.

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

### Checkboxes

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
| `reset`  | Clears all data that a user entered into a form and sets the forms back to their default values. |
| `button` | Generic button that you can use for anything. Commonly used with JS to create interactive UIs. |


> By default, a form button `type` attribute is set to `submit`. Always set the `type` attribute so that the button does not submit the form by accident

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
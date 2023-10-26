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
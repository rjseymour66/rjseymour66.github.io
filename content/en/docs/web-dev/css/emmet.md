---
title: "Emmet for VSCode"
linkTitle: "xEmmet"
weight: 240
# description:
---

Find key bindings with Ctrl + k then Ctrl + s.

## Links

- [Official docs](https://docs.emmet.io/)
- [Emmet cheat sheet](https://docs.emmet.io/cheat-sheet/)
- [Keybindings plugin](https://marketplace.visualstudio.com/items?itemName=agutierrezr.emmet-keybindings)

## Wrapper

```html
.wrapper>h1{Title}+.content
```

```html
<div class="wrapper">
  <h1>Title</h1>
  <div class="content"></div>
</div>
```

## Auto-gen

Creates button:

```html
button[type="button"]
```

### Creates div

```
[data-selected]
```

### Add text to auto-gen elements:

```html
header>nav>ul>li*3{test}
```

```html
<header>
  <nav>
    <ul>
      <li>test</li>
      <li>test</li>
      <li>test</li>
    </ul>
  </nav>
</header>
```

### Dynamic auto-gen:

```html
header>nav>ul>li*3{List Item $}
```

```html
<header>
  <nav>
    <ul>
      <li>List Item 1</li>
      <li>List Item 2</li>
      <li>List Item 3</li>
    </ul>
  </nav>
</header>
```

### Dynamic classes:

```html
header>nav>ul>li*3.class-${List Item $}
```

```html
<header>
  <nav>
    <ul>
      <li class="class-1">List Item 1</li>
      <li class="class-2">List Item 2</li>
      <li class="class-3">List Item 3</li>
    </ul>
  </nav>
</header>
```

### Zero padding

```html
header>nav>ul>li*3.class-${List Item $$}
```

```html
<header>
  <nav>
    <ul>
      <li class="class-1">List Item 01</li>
      <li class="class-2">List Item 02</li>
      <li class="class-3">List Item 03</li>
    </ul>
  </nav>
</header>
```

### Grouping

Group with parentheses:

```html
(header>nav)+main+footer
```

```html
<header>
  <nav></nav>
</header>
<main></main>
<footer></footer>
```

Complex example:

```html
(header>h2{Heading}+nav>li*5>a{Link $})+main+footer
```

```html
<header>
  <h2>Heading</h2>
  <nav>
    <li><a href="">Link 1</a></li>
    <li><a href="">Link 2</a></li>
    <li><a href="">Link 3</a></li>
    <li><a href="">Link 4</a></li>
    <li><a href="">Link 5</a></li>
  </nav>
</header>
<main></main>
<footer></footer>
```

## Forms

```html
form:post>.group>input:text
```

```html
<form action="" method="post">
  <div class="group"><input type="text" name="" id="" /></div>
</form>
```

```html
form:post>(.group>label+input:text)+(.group>label+input:number)
```

```html
<form action="" method="post">
  <div class="group">
    <label for=""></label><input type="text" name="" id="" />
  </div>
  <div class="group">
    <label for=""></label><input type="number" name="" id="" />
  </div>
</form>
```

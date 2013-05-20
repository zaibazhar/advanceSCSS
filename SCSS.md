# N cool things you may not have known about SCSS

## Things you probably knew already...

...but are always worth mentioning, because they're incredibly cool compared to vanilla CSS:

 * [Nested selectors](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#css_extensions)
 * [Variables](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#variables_)
 * [Mixins](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#mixins)

## Prefixing parent selector references

The familiar way:
```scss
a {
    &:hover {
        color: red;
    }
}
```
```css
/* compiled CSS */
a:hover {
  color: red;
}
```
But `&` can be used with a prefix just as well:
```scss
p {
    body.no-touch & {
        display: none;
    }
}
```
```css
/* compiled CSS */
body.no-touch p {
  display: none;
}
```
This can be very useful when you have a deep nesting of rules already in place, and you want to effect a change to the styling of an element based on a selector match much closer to the DOM root.  Client capability flags such as the `no-touch` class are often applied in such a way, to the `<body>` element for example.

## Variable interpolation in selectors

Variables can be expanded in selectors, too:
```scss
$alertClass: "error";

p.message-#{$alertClass} {
    color: red;
}
```
```css
/* compiled CSS */
p.message-error {
  color: red;
}
```
...or almost anywhere else for that matter, like in media queries or CSS comments:
```scss
$breakpoint: 768px; // this would likely go to a _settings.scss somewhere

@media (max-width: #{$breakpoint}) {
    /* This block only applies to viewports <= #{$breakpoint} wide... */
}
```
```css
/* compiled CSS */
@media (max-width: 768px) {
  /* This block only applies to viewports <= 768px wide... */
}
```

## Variable defaults

If your SCSS module can be configured using globals (which tends to be the SCSS way), you can declare them with a default value:
```scss
// _my-module.scss:
$message-color: blue !default;

p.message {
    color: $message-color;
}
```
```css
/* compiled CSS */
p.message {
  color: blue;
}
```
But you can then optionally override the module defaults before its inclusion:
```scss
$message-color: black;
@import 'my-module';
```
```css
/* compiled CSS */
p.message {
  color: black;
}
```
That is, an assignment with a `!default` will only take effect if such a variable didn't have a value before (in contrast to the standard assignment, which will always overwrite a possible previous value).

This is how many SCSS modules (including most that ship with Compass) are configured.

## Control directives

SCSS sports the standard set of flow control directives, such as `if`:
```scss
$debug: false; // this would likely be in a _settings.scss somewhere

article {
    color: black;

    @if ($debug) { // visualizing layout internals
        border: 1px dotted red;
    }
}
```
```css
/* compiled CSS */
article {
  color: black;
}
```
Having such compile-time flags in your project's styling can help debug complex layout issues visually, faster than just inspecting the page an element at a time.

There's also `@for`, `@each` and `@while`.  They're good for a number of things, like:
```scss
@each $name in 'save' 'cancel' 'help' {
    .icon-#{$name} {
        background-image: url('/images/#{$name}.png');
    }
}
```
```css
/* compiled CSS */
.icon-save {
  background-image: url("/images/save.png");
}
.icon-cancel {
  background-image: url("/images/cancel.png");
}
.icon-help {
  background-image: url("/images/help.png");
}
```
...and much more.  Keep in mind, though, that if you need them in your daily styling work you're probably overdoing it.  Instead, they usually warrant the added complexity when building configurable SCSS modules and other such reusable components.

The interested reader can check out the [full documentation on the subject](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#control_directives).

## The list data type

As we saw in the previous example, `@each` can iterate over a list.  Lists are in fact a [fundamental part](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#lists) of the SCSS language, but a quick demo of their application might be configuring some generated styling:
```scss
$buttonConfig: 'save' 50px, 'cancel' 50px, 'help' 100px;

@each $tuple in $buttonConfig {
    .button-#{nth($tuple, 1)} {
        width: nth($tuple, 2);
    }
}
```
```css
/* compiled CSS */
.button-save {
  width: 50px;
}
.button-cancel {
  width: 50px;
}
.button-help {
  width: 100px;
}
```
This demonstrates two features of the list data type, namely the `nth()` [list accessor function](http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html#nth-instance_method), and more interestingly list nestability: in JavaScript notation, the above would be equivalent to:
```js
var buttonConfig = [[ 'save', 50 ], [ 'cancel', 50 ], [ 'help', 100 ]];
```
That is, lists can be separated by both spaces and commas, and alternation between the two notations is also allowed.

## Color arithmetic

TODO

## Math (and other) functions

http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html

## Defining custom functions

No Ruby needed:

```scss
@function make-more-spacious($value) {
    @return $value + 20px;
}

p {
    margin: make-more-spacious(50px);
}
```

Compiles into:

```css
/* compiled CSS */
p {
  margin: 70px;
}
```

## Argument defaults for functions/mixins

```scss
@mixin foobar($a, $b, $padding: 20px) {
    // do something with all these arguments...
}

p {
    @include foobar(123, "abc");
}

p.important {
    @include foobar(123, "abc", 50px);
}
```

## Keyword arguments

If you have a huge helper with:

```scss
@mixin foobar($topPadding: 10px, $rightPadding: 20px, $bottomPadding: 10px, $leftPadding: 20px, $evenMorePadding: 10px) {
    // do something with all these arguments...
}

p {
    @include foobar($bottomPadding: 50px); // specify only the argument you want to override
}
```

Note the `@include mixin() { overrides; }` pattern too.

## Variable arguments for functions/mixins

...and expanding and/or extending during further calls.

## Content block arguments for mixins

```scss
@mixin only-for-mobile {
    @media (max-width: 768px) {
        @content;
    }
}

@include only-for-mobile {
    p {
        font-size: 150%;
    }
}
```

Compiles into:

```css
/* compiled CSS */
@media (max-width: 768px) {
  p {
    font-size: 150%;
  }
}
```

You can mix standard and content block arguments, too:

```scss
@mixin only-for-mobile($breakpoint) {
    @media (max-width: #{$breakpoint}) {
        @content;
    }
}

@include only-for-mobile(480px) {
    p {
        font-size: 150%;
    }
}
```

## Media query bubbling

```scss
p {
    @media (max-width: 768px) {
        // Use larger text for smaller screens:
        font-size: 150%;
    }
}
```

Compiles into:

```css
/* compiled CSS */
@media (max-width: 768px) {
  p {
    font-size: 150%;
  }
}
```

## Media query nesting

```scss
p {
    @media (max-width: 768px) {

        // Use larger text for smaller screens:
        font-size: 150%;

        @media (orientation: landscape) {

            // Condense text a bit because of small vertical space:
            line-height: 75%;
        }
    }
}
```

Compiles into:

```css
/* compiled CSS */
@media (max-width: 768px) {
  p {
    font-size: 150%;
  }
}
@media (max-width: 768px) and (orientation: landscape) {
  p {
    line-height: 75%;
  }
}
```

## Extending classes

...with multiple inheritance.

## Placeholder selectors

TODO

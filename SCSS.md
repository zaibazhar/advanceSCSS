# Advanced SCSS

Or, **16 cool things you may not have known your stylesheets could do.**  I'd rather have kept it to a nice round number like 10, but they just kept coming.  Sorry.

This isn't an [introduction to the language](http://sass-lang.com/tutorial.html) by a long shot; many things probably won't make sense unless you have some SCSS under your belt already.  That said, if you're not yet comfy with the basics, check out the awesome CSS extensions you've always wished you had:

 * [Nested selectors](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#css_extensions)
 * [Variables](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#variables_)
 * [Mixins](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#mixins)
 * and loads more

Also, if you have any questions, comments or corrections, leave a comment below, or fork this gist and let me know!

So without further ado, on to the *cool things*.  The tips are vaguely ordered from basic to more advanced.  If you're immediately bored, feel free to skip ahead to the deep end of the pool.

## 1. Prefixing parent selector references

This is the familiar way you're probably using `&`:
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
This can be very useful when you have a deep nesting of rules already in place, and you want to effect a change to the styling of an element based on a selector match much closer to the DOM root.  Client capability flags such as the [Modernizr](http://modernizr.com/docs/) `no-touch` class are often applied this way to the `<body>` element.

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
$breakpoint: 768px;

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

The media query example is particularly useful if the `$breakpoint` variable is defined in a `_settings.scss`, so the breakpoints of the entire application are configurable from one file.

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
$debug: false; // TODO: move to _settings.scss

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

There's also `@for`, `@each` and `@while`.  They're good for a number of use cases that would otherwise need lots of repetitive (S)CSS, like:
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

The interested reader can check out the [full documentation on control directives](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#control_directives).

## The list data type

As we saw in the previous example, `@each` can iterate over a list.  Lists are in fact a [fundamental part](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#lists) of the SCSS language, but a quick demo of their application might be configuring some generated styling:
```scss
$buttonConfig: 'save' 50px, 'cancel' 50px, 'help' 100px; // TODO: move to _settings.scss

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
That is, lists can be separated by both spaces and commas, and alternation between the two notations produces nested lists.

## Defining custom functions

[Mixins](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#mixins) are a well-known part of the language, but SCSS also allows you to define custom functions.  Contrary to what one might expect, this can also be done in pure SCSS, instead of [extending SCSS in Ruby](http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html#adding_custom_functions):
```scss
@function make-greener($value) {
    @return $value + rgb(0,50,0); // arithmetic with colors is _b
}
p {
    background: make-greener(gray);
}
```
```css
/* compiled CSS */
p {
  background: #80b280;
}
```
The above color is a gray with a slight green tint.

Functions are most useful in avoiding some repeated computation in an expression.  It also implicitly documents that computation by giving it a name: the above example, while contrived, is still more understandable than just having arbitrary color arithmetic in the style block.  SCSS ships with a ton of [useful built-in functions](http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html), and Compass [adds even more](http://compass-style.org/reference/compass/helpers/), so do first check whether there's a built-in equivalent before implementing your own helper.

## Argument defaults

Arguments and functions support default values for arguments; the *last* zero-to-N arguments can be made optional by providing them with a default value:

```scss
@mixin foobar($a, $b, $padding: 20px) {
    padding: $padding;
    // ...and something with $a and $b
}

p {
    @include foobar(123, "abc"); // the default padding's fine
}

p.important {
    @include foobar(123, "abc", 50px); // override the default
}
```

## Keyword arguments

If your mixin (or function) takes *a lot* of arguments, there's a similar call-time syntax for selecting only specific arguments to override:

```scss
@mixin foobar($topPadding: 10px, $rightPadding: 20px, $bottomPadding: 10px, $leftPadding: 20px, $evenMorePadding: 10px) {
    // do something with all these arguments...
}

p {
    @include foobar($bottomPadding: 50px); // specify only the argument you want to override
}
```

However, in cases where a lot of the arguments are just for overriding specific CSS properties (such as `top-padding`, `bottom-padding` and so on), the "content block overrides -pattern" is likely a better fit (see below).

## Variable arguments for functions/mixins

[Var-args](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#variable_arguments) work much the same way as in other languages that support the feature; any extra arguments to a function/mixin call are wrapped into a list and assigned to the argument having a `...` suffix:

```scss
@mixin config-icon-colors($prefix, $colors...) {
    @each $i in $colors {
        .#{$prefix}#{nth($i, 1)} {
            color: nth($i, 2);
        }
    }
}
@include config-icon-colors('icon-',
    'save'   green,
    'cancel' gray,
    'delete' red
);
```
```css
/* compiled CSS */
.icon-save {
  color: green;
}
.icon-cancel {
  color: gray;
}
.icon-delete {
  color: red;
}
```

The above helper could be used to set up colors for your font icons, [Font Awesome](http://fortawesome.github.io/Font-Awesome/) for example, without having to repeat yourself.  The helper works by passing in a variable number of arguments (after the first, required one).  Each of those arguments is expected to be a tuple of two items (again in JavaScript notation, for example `[ "save", "green" ]`).

In fact, the `...` syntax also works during call-time, where it expands a list into separate arguments, to be fed into the target mixin:

```scss
@mixin foobar($a, $b, $c) {
    // receives args $a = 5px, $b = red, and so on
}

$myArgs: 5px red "bla bla";
// at this point, you could also programmatically add/remove arguments

@include foobar($myArgs...);
```

Personally, I have yet to find a use case for this, but [the documentation](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#variable_arguments) has a nice use case for passing current arguments forward to another mixin.

## Content block arguments for mixins

Since [version 3.2.0](http://sass-lang.com/docs/yardoc/file.SASS_CHANGELOG.html#320_10_august_2012), SCSS has had an implicit mixin argument accessible through the `@content` directive.  It allows passing an entire SCSS content block as an argument to the mixin:

```scss
@mixin only-for-mobile {
    @media (max-width: 768px) {
        @content;
    }
}

@include only-for-mobile() /* note: @content begins here */ {
    p {
        font-size: 150%;
    }
} /* @content ends */
```
```css
/* compiled CSS */
@media (max-width: 768px) {
  p {
    font-size: 150%;
  }
}
```

This is a very powerful language feature.  You can mix standard and content block arguments, too:

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

## Content block overrides -pattern

Consider a mixin that generates a selector and a corresponding style block, allowing the caller to customize the styling if needed:
```scss
@mixin message($class, $color: yellow, $margin: 20px, $padding: 10px) {
    .message-#{$class} {
        border: 1px dotted $color;
        color: $color;
        margin: $margin;
        padding: $padding;
    }
}
```
Calling this mixin using keyword arguments (see above) is quite convenient, because if the defaults are fine, no extra arguments need be provided.  If they're not, you only need to specify the arguments you want to override:
```scss
@include message("subtle", $margin: 5px);
```
But this requires you to list *all overridable properties* in the mixin signature.  However, content block arguments allow arbitrary overrides without the argument jungle:
```scss
@mixin message($class) {
    .message-#{$class} {
        border: 1px dotted yellow;
        color: yellow;
        margin: 20px;
        padding: 10px;
        @content;
    }
}

@include message("subtle") {
    margin: 5px;
}
```
```css
/* compiled CSS */
.message-subtle {
  border: 1px dotted yellow;
  color: yellow;
  margin: 20px;
  padding: 10px;
  margin: 5px;
}
```
Here, we allow the good-ole CSS cascade to effect the property override (the latter `margin` overrides the former one).  Also, we're not limited to overriding the properties the author of the mixin thought of.  In fact, this allows passing in nested blocks as well:
```scss
@include message("actionable") {
    button {
        float: right;
    }
}
```
```css
/* compiled CSS */
.message-actionable {
  border: 1px dotted yellow;
  color: yellow;
  margin: 20px;
  padding: 10px;
}
.message-actionable button {
  float: right;
}
```
This pattern can be useful in any library code that outputs nontrivial styling with generated selectors; in simple cases (where no selectors are emitted within the mixin) it's of course rather unnecessary, as any overrides can be made directly after the mixin call.

## Media query bubbling

`@media` blocks do not need to be declared at the root level of the stylesheet:
```scss
p {
    @media (max-width: 768px) {
        font-size: 150%; // use larger text for smaller screens
    }
}
```
```css
/* compiled CSS */
@media (max-width: 768px) {
  p {
    font-size: 150%;
  }
}
```
Notice how the compiler "bubbles up" the `@media` block to the root level (since regular CSS doesn't support selector nesting), and then outputs within all styling and the corresponding selectors that were encountered within the `@media` block in the SCSS source.

This is very useful as it allows you to make media-specific tweaks almost anywhere in your styling, right where they're relevant, instead of collecting all those tweaks to the end of the stylesheet, and hoping that their selectors will stay in sync with the ones they're overriding (they won't).

## Media query nesting

The aforementioned bubbling mechanism also takes nesting into account, and combines all applicable queries with the `and` media query operator:
```scss
p {
    @media (max-width: 768px) {
        font-size: 150%; // use larger text for smaller screens
        @media (orientation: landscape) {
            line-height: 75%; // condense text a bit because of small vertical space
        }
    }
}
```
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

## Extending selectors

SCSS allows [extending selectors](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#extend), by copying and combining selectors in the CSS output.  Interestingly, while the mechanism is (obviously) very different, the semantics of `@extend` are quite analogous to traditional object-oriented programming languages (such as Java & whatnot):
```scss
.animal {
    background: gray;
}
.cat {
    @extend .animal;
    color: white;
}
```
```css
/* compiled CSS */
.animal, .cat {
  background: gray;
}
.cat {
  color: white;
}
```
That is, `.cat` has all the properties of its "parent class" `.animal`, plus any specific ones it adds or overrides.  Whereas in normal CSS you would have to reference both the extending class and the parent class (e.g. `<div class="animal cat">`), now you can only name the exact class you want (`<div class="cat">`).  What it does (or doesn't) inherit from depends on the definition of `.cat`.

Classical inheritance, right?  Overriding properties in the "child class" works due to the style cascade in the browser: styling for the same selector that comes later in the file always wins over the styling that came before it.  Perhaps a bit unintuitively, this actually works out fine even if the combined selectors have differing [specificity](http://www.w3.org/TR/css3-selectors/#specificity) (think `.class` overriding an `#id`).  Extending selectors may often be preferable to using mixins to achieve the same effect:
```scss
@mixin animal {
    background: gray;
    border: 1px solid red;
    font-weight: bold;
    font-size: 50px;
    color: red;
    padding: 20px;
}
.cat {
    @include animal;
    color: white;
}
.dog {
    @include animal;
    color: black;
}
```
```css
/* compiled CSS */
.cat {
  background: gray;
  border: 1px solid red;
  font-weight: bold;
  font-size: 50px;
  color: red;
  padding: 20px;
  color: white;
}
.dog {
  background: gray;
  border: 1px solid red;
  font-weight: bold;
  font-size: 50px;
  color: red;
  padding: 20px;
  color: black;
}
```
Notice how only the last property (`color`) is different.  As we define more types of animals, the amount of repeated style properties keeps growing.  This is in contrast to how selector extension would solve the same problem:
```scss
.animal {
    background: gray;
    border: 1px solid red;
    font-weight: bold;
    font-size: 50px;
    color: red;
    padding: 20px;
}
.cat {
    @extend .animal;
    color: white;
}
.dog {
    @extend .animal;
    color: black;
}
```
```css
/* compiled CSS */
.animal, .cat, .dog {
  background: gray;
  border: 1px solid red;
  font-weight: bold;
  font-size: 50px;
  color: red;
  padding: 20px;
}
.cat {
  color: white;
}
.dog {
  color: black;
}
```
Finally, selector extension allows for integrations into 3rd party CSS libraries, that need not be specifically designed for extension, or even be written in SCSS.  [Twitter Bootstrap](http://twitter.github.io/bootstrap/), for example, includes nice styling for buttons, but doesn't apply it to `<button>` elements by default.  In our quest to reduce unnecessary CSS classes, we can fix this simply with:
```scss
@import "bootstrap.scss"; // just a renamed .css file, so that @import works

button {
    @extend .btn;
}
```
This will add the `button` selector into the Bootstrap source wherever `.btn` is referenced.

## Placeholder selectors

Because in the above example(s) the `.animal` base class isn't used anywhere directly (only through its child classes), we might just as well get rid of it in the CSS output.  SCSS allows this with [placeholder selectors](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#placeholders) - whereas `.foo` signifies a class, and `#foo` an ID, `%foo` is considered a placeholder, and gets special treatment by the parser: its styles are never output on their own, only through extension.
```scss
%animal {
    background: gray;
    // and so on...
}
.cat {
    @extend %animal;
    color: white;
}
.dog {
    @extend %animal;
    color: black;
}
```
```css
/* compiled CSS */
.cat, .dog {
  background: gray;
}
.cat {
  color: white;
}
.dog {
  color: black;
}
```
Because `%animal` was just a placeholder selector, it's disappeared from the output.  As an added benefit, if you never define a selector that extends `%animal`, its styles (`background: gray;` and so on) are completely omitted.  This can be very useful in SCSS framework authoring, as you can offer any number of base classes for opt-in extension, but only the ones actually *used* are output into the resulting CSS.

Placeholder selectors can actually do even more than this, namely expanding the `%placeholder` part into a more complex selector during `@extend`, but personally I've never had to use this feature.  The interested reader can check out the [docs on the subject](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#placeholders).

## Selector multiple inheritance

A selector can actually inherit from *several* other selectors - that is, SCSS supports multiple inheritance.  For each `@extend`, the current selector is appended to the selector being extended.  When combined with placeholder selectors, this allows powerful abstractions for styling framework authors.  This is perhaps best explained through an example:
```scss
// in the framework files:
%mfw-standing-out {
    font-size: 150%;
    font-style: italic;
    padding: 25px;
}
%mfw-slightly-shadowed {
    @include box-shadow(black 2px 2px 10px); // from Compass
}
%mfw-rounded {
    @include border-radius(25px); // from Compass
}

// in the application files:
#join-button {
    @extend %mfw-standing-out;
    @extend %mfw-slightly-shadowed;
    @extend %mfw-rounded;
    background: green;
    color: white;
}
```
This way of constructing styling has a few notable benefits:

 * **Self-documentation:** Instead of writing out loads of anonymoud style properties, the author instead describes the "traits" of the UI component he is designing.
 * **Naming isolation:** The application can use its own naming conventions and semantics in the HTML, while the framework naming conventions never make their way into the compiled CSS.  In the above example, the framework adopts a common prefix `%mfw` (for "my framework", or whatever) to avoid naming collisions with other SCSS libraries.
 * **Reduced repetition:** The `#join-button` could use the `border-radius()` and `box-shadow()` helpers directly to achieve the same stylistic effect.  But for each additional component that does so, the `box-shadow()` helper would output the exact same lines of CSS, with all the vendor prefixes and whatnot.  Extending `%mfw-slightly-shadowed`, however, would simply append the selector to the list of other selectors that should receive shadowing.
 * **Opt-in:** If a specific "trait" is never used to describe some UI component, its styles are never output.  That is, if you just use 1% of the features of a bloated style framework, your CSS payload will only contain that 1%.  Contrast this to the de-facto way of just including an entire Bootstrap or jQuery, because the site uses 1 or 2 features from each.

Potential downsides with this approach should be kept in mind:

 * **Debuggability:** Without properly configured [source maps for SCSS](http://bricss.net/post/33788072565/using-sass-source-maps-in-webkit-inspector), it may be harder to figure out how some styling ended up affecting a specific element (keep in mind the above point about *self-documentation* no longer applies in the compiled CSS).
 * **Arguments:** This technique won't replace mixins when computation is required based on some arguments.  While there can be several versions of each "trait" (think `%mfw-slightly-shadowed` vs `%mfw-heavily-shadowed`), they'll always be completely static in content.

...and that's it!

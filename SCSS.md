# N cool things you may not have known about SCSS

## Things you probably knew already...

...but are always worth mentioning, because they're incredibly cool compared to vanilla CSS:

 * [Nested selectors](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#css_extensions)
 * [Variables](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#variables_)
 * [Mixins](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#mixins)

## Prefixing parent selector references

Familiar way:

```scss
a {
    &:hover {
        color: red;
    }
}
```
Compiles into:
```css
/* compiled CSS */
a:hover {
  color: red;
}
```

But can be used with a prefix just as well:

```scss
p {
    body.no-touch & {
        display: none;
    }
}
```

Gives you:

```css
/* compiled CSS */
body.no-touch p {
  display: none;
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

## Control directives

```scss
$debug: false; // this would likely be in a _settings.scss somewhere

article {

    color: black;

    @if ($debug) {
        border: 1px dotted red;
    }
}
```

Compiles into:

```css
/* compiled CSS */
article {
  color: black;
}
```

There's also `@for`, `@each` and `@while`.

## The list data type

TODO

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

p {
    @include only-for-mobile {
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

p {
    @include only-for-mobile(768px) {
        font-size: 150%;
    }
}
```

## Variable interpolation in selectors

```scss
$alertClass: "error";

p.message-#{$alertClass} {
    color: red;
}
```

Compiles into:

```css
/* compiled CSS */
p.message-error {
  color: red;
}
```

...or almost anywhere else, for that matter.

```scss
$breakpoint: 768px; // this would likely go to a _settings.scss somewhere

@media (max-width: #{$breakpoint}) {
    /* This block only applies to viewports <= #{$breakpoint} wide... */
}
```

Compiles into:

```css
/* compiled CSS */
@media (max-width: 768px) {
  /* This block only applies to viewports <= 768px wide... */
}
```

## Variable defaults

## Extending classes

...with multiple inheritance.

## Placeholder selectors


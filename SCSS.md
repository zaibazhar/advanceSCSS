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

## Media query nesting

## The list data type

## Color arithmetic

## Defining custom functions

## Keyword arguments

## Variable interpolation in selectors

...or almost anywhere else, for that matter.

## Variable defaults

## Extending classes

...with multiple inheritance.

## Control directives

## Argument defaults for functions/mixins

## Variable arguments for functions/mixins

...and expanding and/or extending during further calls.

## Content block arguments for mixins

## Math (and other) functions

## Placeholder selectors

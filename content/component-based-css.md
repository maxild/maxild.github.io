+++
title = "Component-based CSS"
date = 2020-07-08
[taxonomies]
tags = ["styling"]
+++

The following text is copied from some old notes of mine. It is all about
BEM, OOCSS, SMACSS etc.

<!-- more -->

```html
<body>
  <!-- http://outdatedbrowser.com/ could also be used -->

  <!--[if lte IE 9]>
    <p class="browserupgrade">You are using an <strong>outdated</strong> browser. Please <a href="https://browsehappy.com/">upgrade your browser</a> to improve your experience and security.</p>
  <![endif]-->

  <!-- More content here off course -->

</body>
```

> As of Internet Explorer 10, conditional comments are no longer supported by standards mode. Use feature detection to provide effective fallback strategies for website features that aren't supported by the browser.

## Misc

* Naming css classes can be read about in this [post](https://css-tricks.com/semantic-class-names/)
* Use pseudo BEM for naming conventions. Check this [link](https://rscss.io/)...maybe it is irrelevant.
* Nesting in Sass looks cleaner, but I think there’s a definite drawback in not being able to CTRL+F (grep) for something inside the codebase. Also makes diff's nicer, and PR's on github.
* BEM (and other component methodologies) promote a one-component-per-file approach. This makes the search easier.
* See [this](https://github.com/danielguillan/bem-constructor) github project and [this](https://codepen.io/andersschmidt/post/expressive-bem-with-sass-a-different-approach) blog post about using mixins to express BEM naming conventions.

## Pre-processor stuff (tools layer)

* setting
* mixins

## Generic (star and element selectors)

Thin layer

* normalize.css
* resets (maybe)
* box-sizing

## Base (element selectors)

Thin layer. Maybe normalize is enough. Basics that doesn't change from project to project.

```css
table { width: 100%; }

body, form {
  margin: 0;
  padding: 0;
}
```

## Utilities

* ```u-utilityName``` (trump layer in ITCSS, utility classes are quaick and dirty way of doing stuff)

```css
.text-xl { font-size: 140%; }
.text-l { font-size: 120%; }
.text {}
.text-s { font-size: 90%; }
.text-xs { font-size: 80%; }
```
maybe shoulf be ```u-text-xl``` in the markup.

## Layout

Mini layout problems with a horizontal list of buttons. below are a few different solutions (owl selector and modifiers)

```css
* + * { margin-left: 5px; }
/* this selector will keep the HTML simple, and put the burden on the more fragile combinator selector */
.button + .button { margin-left: 5px; }
/* The following selectors will put the burden on the HTML, but decouple the CSS from the markup structure */
.button--first { }
.button--last { }
```

## Components

Maybe differentiate **layout** into own file, but do not prefix a la hungarian notaion. Layout is about containers and size and positioning (specifying ```width``` and ```margin```). The site logo and/or main navigation: ```.logo {}``` and ```.main-nav {}```. are somehow “reusable patterns”, but they're unique in the sense that that they only appear once on any given page and therefore (serverside) are rendered by the layout template.

The big layer is components. Components expand to fill the layout. This way components and layout are different.

Accord­ing to the BEM model, we should name these using the containing block name, two underscores, and then the element name.

* ```{block-name}``` (component, module)

Some components are variations of their "base" component, so they are marked up with the suffix ```--modifier-name```.

* ```{block-name}--{modifier-name}``` (component/module variation)

The modifier class is a separate class that is declared on its own. That is ```btn``` and ```btn--large``` er two different classes that both are used on a large button in the HTML.

In general you should favour the multiple-class approach over using something like ```@extend``` in Sass (just to make a single class possible in the HTML). Using multiple classes in your markup

* gives you a better paper-trail (documenation) in your markup, and allows you to see quickly and explicitly which classes are acting on a piece of HTML
* allows for greater composition in that classes are not tightly bound to other styles in your CSS.

Whenever you are building a UI component, try and see if you can break it into two parts: one for structural/base styles (paddings, layout, etc.) and another for skin/variations (colors, typefaces etc.).

An element is a descendant element of the block element that is scoped to the block. It is like a "value object" of the component that acts like an "aggregate root". It acts like a flattened class selector that makes the styling more explicit and lowers specificity because (long) descendant combinators are avoided (that is nesting in Sass should also be avoided).

* ```{block-name}__{descendent-name}``` (child component)

Element names do not reflect the block’s structure. Regardless of nested levels within, it’s always just the block-name and the descendent-name (so, never .block__elem1__elem2, just block__elem).

The state prefix ```is-``` identifies that javascript is coming in here to toggle (add or remove) the state of the UI component. Also remember to use javascript as a state machine manipulating/toggling the state classes in the CSS. Do not use javascript to write (inline) styles to DOM elements. For instance do not use jQuery ```show()```/```hide()``` methods.

* ```{block-name}.is-{state-of-component}```
* ```{block-name}.has-{state-of-component}```

Use ```is-{state-of-component}``` to reflect changes to a component's (temp­orary) state (e.g. is-selected, is-active, is-disabled, is-hovered). **Never style these classes directly; they should always be used as an adjoining class (sometimes also called class chaining).** Maybe states could also be ```btn-is-disabled``` for more specific states (thereby flattening the selector and lowering specificity). This is how it is presented in SMACSS:

> States that are specific to a module (component, block) should include module name.

But you may have state that is generic like ```is-hidden``` (or ```visually-hidden``` see [htmlboilerplate-normalize](https://gist.github.com/necolas/1024797#file-style-css-L196-L222)), but maybe this is a utility class.

```scss
button {
    // Forbid interaction with buttons that are disabled.
    &[disabled],
    &.is-disabled {
        opacity: 0.5;
        pointer-events: none;
    }
}
```

So are all selectors single class selectors of equal specificity? Well, almost. In some cases, you might need two class names in a selector — for example, when a block modifier affects individual elements:

```css
.text-input--disabled .text-input__label {
   display: none;
}
```

The nice thing is that any rule that redefines this one will likely depend on another modifier (because of the unified semantics!), which means that specificity is still the same and only the rule (source) order matters.

## Patterns

* Layout specify width and margin
* Components expand to fill layout (fluid, responsive)
* Layout and components should **not** use ```!important```
* States are ```!important``` (you may need to throw in 'bang-important' for states, because state is just a single class name, and may need to override something with higher specificity...but put states in the trumps layer. Example ```a::hover``` will win over ```.is-hovered``` therefore we need to add ```!important```)

## Hungarian notation on classes/selectors

* ```u-``` for utility class (those grubby little styles that have ```!important``` after them -- aka trumps)
* ```l-``` for layout
* ```c-``` for components/modules
* ```o-``` for objects (OOCSS: objects are sort of like generic components/modules).
* ```is-``` or ```has-``` for state.
* ```js-``` for classes which influence the behav­iour of the element through javascript. Probably should be only used by javascript. That is never styled, so no css on these classes. Only used by javascript to bind event handlers or what not -- ```document.getElementsByClassName```.

I don't think I like hungarian notation, besides using ```u-``` on utility classes, because I don't like utility classes in the first place.

## Alternative naming conventions

Some alternative naming conventions are simpler than BEM, meaning less noise, less hyphens and no(!) underscores in the HTML and CSS.

### SUITCSS (structured class names and meaningful hyphens. Also check [this](http://nicolasgallagher.com/about-html-semantics-front-end-architecture/) blog post)

Here we use camelCase:

* ```componentName```
* ```componentName--modifierName```: same as BEM
* ```componentName-descendentName```:  no underscores, hyphen signals child element

Here we use PascalCase (this is the naming convention of SUITCSS):

* ```ComponentName``` (root node)
* ```ComponentName--modifierName``` (variation)
* ```ComponentName-descendentName``` (child element)

### SMACSS v2 (with better naming convention, where variation and child element are differentiated in the CSS)

Here we reverse the meaning of `-` and `--`:

* ```moduleName``` (root node)
* ```moduleName-modifierName``` (variation)
* ```moduleName--descendentName``` (child element)

> There is very little difference between BEM and SMACSS. In SMACSS, it talks of modules, sub-modules, and sub-components. These map directly to blocks, modifiers, and elements. (Jonathan Snook)

## Semantic naming

> Keep using agnostic, abstract, OOCSS classes in your markup, but add any desired meaning to your HTML via a ```data-ui-component``` attribute, e.g.: ```<ul class="ui-list" data-ui-component="users-list">```. For more information see this [article](https://csswizardry.com/2014/03/naming-ui-components-in-oocss/), and also read this great [article](http://nicolasgallagher.com/about-html-semantics-front-end-architecture/) by Nicolas Gallagher.

## SRP and OOCSS

If subsets of a style rule can be split out into more manageable and reusable abstractions use single responsibility principle (SRP). Any time you are coding a potentially repeatable design pattern then try and refactor it out into smaller reusable (object, utility, helper) classes .

It’s important not to take this too far; classes should be abstracted but ideally not presentational. Classes like ```.round-corners``` for the sake of SRP are really not all that advisable. Instead use ```@mixin``` to combat DRY.

HTML:

```html
<div class="box content">
    <form>
        <p class="box is-error">Please provide your name.</p>
        <label for="name" class="is-error">Name:</label>
      <input name="name"></input>
    </form>
</div>
```

CSS:

```css
.content {
    width: 640px;
    float: left;
    margin-right: 20px;
}

.box {
    display: block;
    padding: 20px;
    margin-bottom: 20px;
}

.is-error {
    background-color: #fce0e2;
    color: #c00;
}

.is-error.box {
    border: 2px solid #c00;
    border-radius: 8px;
}
```

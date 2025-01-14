---
title: Styling templates
eleventyNavigation:
  key: Styling templates
  parent: lit-html
  order: 4
versionLinks:
  v2: components/styles/
---

lit-html focuses on one thing: rendering HTML. How you apply styles to the HTML lit-html creates depends on how you're using it—for example, if you're using lit-html inside a component system like LitElement, you can follow the patterns used by that component system.

In general, how you style HTML will depend on whether you're using shadow DOM:

*   If you aren't using shadow DOM, you can style HTML using global style sheets.
*   If you're using shadow DOM (for example, in LitElement), then you can add style sheets inside the shadow root.

To help with dynamic styling, lit-html provides two directives for manipulating an element's `class` and `style` attributes:

*   [`classMap`](/docs/v1/lit-html/template-reference/#classmap) sets classes on an element based on the properties of an object.
*   [`styleMap`](/docs/v1/lit-html/template-reference/#stylemap) sets the styles on an element based on a map of style properties and values.

## Setting classes with classMap {#classmap}

Like `styleMap`, the `classMap` directive lets you set a group of classes based on an object.

```js
import {html} from 'lit-html';
import {classMap} from 'lit-html/directives/class-map.js';

const itemTemplate = (item) => {
  const classes = {selected: item.selected};
  return html`<div class="menu-item ${classMap(classes)}">Classy text</div>`;
}
```

More information: see [classMap](/docs/v1/lit-html/template-reference/#classmap) in the Template syntax reference.

## Inline styles with styleMap {#stylemap}

You can use the `styleMap` directive to set inline styles on an element in the template.

```js
import {html} from 'lit-html';
import {styleMap} from 'lit-html/directives/style-map.js';

...

const myTemplate = () => {
  styles = {
    color: myTextColor,
    backgroundColor: highlight ? myHighlightColor : myBackgroundColor,
  };

  return html`
    <div style=${styleMap(styles)}>
      Hi there!
    </div>
  `;
};
```

More information: see [styleMap](/docs/v1/lit-html/template-reference/#stylemap) in the Template syntax reference.

## Rendering in shadow DOM

When rendering into a shadow root, you usually want to add a style sheet inside the shadow root to the template, so you can style the contents of the shadow root.

```js
html`
  <style>
    :host { ... }
    .test { ... }
  </style>
  <div class="test">...</div>
`;
```

This pattern may seem inefficient, since the same style sheet is reproduced in each instance of an element. However, the browser can deduplicate multiple instances of the same style sheet, so the cost of parsing the style sheet is only paid once.

A new feature available in some browsers is <a href="https://wicg.github.io/construct-stylesheets/" target="_blank" rel="noopener">Constructable Stylesheets Objects</a>. This proposed standard allows multiple shadow roots to explicitly share style sheets. LitElement uses this feature in its [static `styles` property](/docs/v1/components/styles/#add-styles).

### Bindings in style sheets

Binding to values in the style sheet is an antipattern, because it defeats the browser's style sheet optimizations. It's also not supported by the ShadyCSS polyfill.

```js
// DON'T DO THIS
html`
  <style>
    :host {
      background-color: ${themeColor};
    }
  </style>
`;
```

Alternatives to using bindings in a style sheet:

*   Use <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/--*" target="_blank" rel="noopener">CSS custom properties</a> to pass values down the tree.
*   Use bindings in the `class` and `style` attributes to control the styling of child elements.

See [Inline styles with styleMap](#stylemap) and [Setting classes with classMap](#classmap) for examples of binding to the `style` and `class` attributes.


### Polyfilled shadow DOM: ShadyDOM and ShadyCSS

If you're using shadow DOM, you'll probably need to use polyfills to support older browsers that don't implement shadow DOM natively. <a href="https://github.com/webcomponents/polyfills/tree/master/packages/shadydom" target="_blank" rel="noopener">ShadyDOM</a> and <a href="https://github.com/webcomponents/polyfills/tree/master/packages/shadycss" target="_blank" rel="noopener">ShadyCSS</a> are polyfills, or shims, that emulate shadow DOM isolation and style scoping.

The lit-html `shady-render` module provides necessary integration with the shady CSS shim. If you're writing your own custom element base class that uses lit-html and shadow DOM, you'll need to use `shady-render` and also take some steps on your own.

The <a href="https://github.com/webcomponents/polyfills/tree/master/packages/shadycss#usage" target="_blank" rel="noopener">ShadyCSS README</a> provides some directions for using shady CSS. When using it with `lit-html`:

*   Import `render` and `TemplateResult` from the `shady-render` library.
*   You **don't** need to call `ShadyCSS.prepareTemplate`.  Instead pass the scope name as a render option. For custom elements, use the element name as a scope name. For example:

    ```js
    import {render, TemplateResult} from 'lit-html/lib/shady-render';

    class MyShadyBaseClass extends HTMLElement {

      // ...

      _update() {
        render(this.myTemplate(), this.shadowRoot, { scopeName: this.tagName.toLowerCase() });
      }
    }
    ```

    Where `this.myTemplate` is a method that returns a `TemplateResult`.

*   You **do** need to call `ShadyCSS.styleElement` when the element is connected to the DOM, and in case of any dynamic changes that might affect custom property values.

	For example, consider a set of rules like this:
    ```js
    my-element { --theme-color: blue; }
	main my-element { --theme-color: red; }
    ```

	If you add an instance of `my-element` to a document, or move it, a different value of `--theme-color` may apply. On browsers with native custom property support, these changes will take place automatically, but on browsers that rely on the custom property shim included with shadyCSS, you'll need to call `styleElement`.

    ```js
    connectedCallback() {
      super.connectedCallback();
      if (window.ShadyCSS !== undefined) {
          window.ShadyCSS.styleElement(this);
      }
    }
    ```


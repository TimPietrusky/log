## forms

```html
<form @submit="${e => this.handleSubmit(e)}">

</form>
```

```js
handleSubmit(e) {
  // Prevent sending data to server
  e.preventDefault()

  // Get data out of the form
  const data = new FormData(e.target)

  const type = data.get('type')
  const amount = parseInt(data.get('amount'), 10)
}
```

## lit-element

### Imports

```js
import { LitElement, html } from '/node_modules/@polymer/lit-element/lit-element.js'
import { repeat } from '/node_modules/lit-html/directives/repeat.js'
import { connect } from 'pwa-helpers/connect-mixin.js'
import { store } from '../../reduxStore.js'
```

### ternary operator
```js
${
  live 
  ? html`<h1>this is live</h1>`
  : ''
}
```


### loop without ID if you need the index

```js
render() {
  const itemTemplates = []

  for (let index = 0; index < mapping.length; index++) {
    const element = mapping[index]

      itemTemplates.push(html`
        ${element} ${index}
      `)
  }

  html`
    ${itemTemplates}
  `
}

```

ORRRRRR

```js
${repeat(animations, animationId => animationId, (animationId, index) => html`
  <animation-list-item class="item" .animation="${getAnimation(store.getState(), { animationId })}"></animation-list-item>
  <button @click="${e => this.handleRemoveAnimation(e)}" .animationIndex="${index}">x</button>
`)}
```

### Force changed value

#### Array

```js
static get properties() {
  return {
    mapping: { 
      type: Array,
      hasChanged: (newValue, oldValue) => { return true }
    },
  }
}
```


## Shared Styles

```js
import { html } from '/node_modules/@polymer/lit-element/lit-element.js'

export const tabs = html`
<style>

</style>
`
```


## Own component with slot and styling from host

```js
import { LitElement, html } from '/node_modules/@polymer/lit-element/lit-element.js'
import '/node_modules/@polymer/paper-button/paper-button.js'

class LuminaveButton extends LitElement {

  render() {
    return html`
        <style>
          :host(.primary) paper-button {
            background-color: var(--primary-background-color);
            color: var(--text-primary-color);
          }
        </style>

        <paper-button>
          <slot></slot>
        </paper-button>
    `
  }
}
```


## Funny errors

If you want to use `e.target.animationIndex` after you set an attibute like this:

```
<button animation-index="${index}"></button>
```

it will not work, it will be under `e.target.attributes`. You have to do it like this

```
<button .animationIndex="${index}"></button>
```
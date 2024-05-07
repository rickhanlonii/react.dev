---
title: useActionState
---

<NextMajor>

The `useActionState` Hook is currently is available in React 19 beta, and the latest React Canary.

In earlier React Canary versions, this API was part of React DOM and called `useFormState`.

Learn more about [React's release channels here](/community/versioning-policy#all-release-channels).

</NextMajor>

<Intro>

`useActionState` is a Hook to handle common use cases for Actions.

```js
const [state, action, isPending] = useActionState(fn, initialState, permalink?);
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useActionState(fn, initialState, permalink?)` {/*useactionstate*/}

Pass a function to `useActionState` to create an Action:

```js
import {updateQuantity} from './api.js';
  
function CheckoutForm() {
  const [state, action, isPending] = useActionState(async (previousState, event) => {
    const savedQuantity = await updateQuantity(event.target.value);
    return savedQuantity;
  }, 1);
  
  return (
    <div>
      <input type="number" name="quantity" defaultValue={state} onChange={action} />
      <button type="submit">Update Quantity</button>
      <p>{isPending ? 'Updating...' : ''}</p>
    </div>
  );
}

```

`useActionState` wraps the function passed to it and returns an Action that calls the function when invoked, the last return value of function, and a boolean indicating whether the Action is pending.

[See more examples below.](#usage)

#### Parameters {/*parameters*/}

* `fn`: Can be an existing Action, or a new Action created in-line. `useActionState` will return a new `action` that calls the passed function as an Action. When the function is called, it will receive the previous state of the Action and the arguments the Action is called with.
* `initialState`: The value you want `useActionState` to return as `state` initially. This argument is ignored after the Action is first invoked.
* **optional** `permalink`: When the Action returned by `useActionState` is passed to a `<form>` element, React will insert the `permalink` string in the form's `action` attribute and submit to that URL if the form is submitted before JavaScript has loaded. This is useful for server-rendered forms that need to be interactive before JavaScript has loaded.

{/* TODO T164397693: link to serializable values docs once it exists */}

#### Returns {/*returns*/}

`useActionState` returns an array with exactly three values:

1. `state`: The current state of the Action. During the first render, it will equal the `initialState` you have passed. After the `action` is invoked, it will match the value returned by the Action.
2. `action`: A new action that you can call. The function passed to `useActionState` will be called when this action is invoked. The arguments passed to the action will be passed to the function as well.
3. `isPending`: A boolean that is `true` while the Action is pending, and `false` otherwise. This is useful for showing pending UI while the Action is in progress.

#### Caveats {/*caveats*/}

* When used with a framework that supports React Server Components, `useActionState` lets you make forms interactive before JavaScript has executed on the client. When used without Server Components, it is equivalent to component local state.
* The function passed to `useActionState` receives an extra argument, the previous or initial state, as its first argument. This makes its signature different than if it were used directly as a form action without using `useActionState`.
* If used with a Server Action, `useActionState` allows the server's response from submitting the form to be shown even before hydration has completed.
* It can be any serializable value.
* if `fn` is a [server action](/reference/rsc/use-server) and the form is submitted before the JavaScript bundle loads, the browser will navigate to the specified permalink URL, rather than the current page's URL. Ensure that the same form component is rendered on the destination page (including the same action `fn` and `permalink`) so that React knows how to pass the state through. Once the form has been hydrated, this parameter has no effect.
---

## Usage {/*usage*/}

### Creating an Action with `useActionState` {/*creating-an-action-with-useactionstate*/}

You can create an Action by passing a function to `useActionState`:

```js [[1, 5, "fn"], [2, 5, "state"], [2, 5, "initialState"], [3, 5, "action"], [4, 5, "isPending"]]
import { useActionState } from "react";
import { updateQuantity } from "./api";

function CheckoutForm() {
  const [state, action, isPending] = useActionState(fn, initialState);
  // ...
}
```

To create an action, pass a <CodeStep step={1}>function</CodeStep> to `useActionState`, which returns:
- The last <CodeStep step={2}>state</CodeStep> returned by the Action. Initially the state is set to the <CodeStep step={2}>initialState</CodeStep> provided.
- The <CodeStep step={3}>action</CodeStep> created by `useActionState`. This is the Action you will call.
- The <CodeStep step={4}>isPending</CodeStep> state of the Action.

You can call the action the same as any other function, such as in an event handler:

```js [[1, 3, "previousState"], [2, 3, "event"], [3, 7, "1"], [4, 2, "action"], [4, 13, "action"]]
// Create the action.
const [state, action, isPending] = useActionState(
  async function onQuantityChange(previousState, event) {
    const savedQuantity = await updateQuantity(event.target.value);
    return savedQuantity;
  },
  1, // initial state
);

//...

// Use the action in an event handler.
<input type="number" onChange={action} />
```

The arguments passed to the Action include:
- The <CodeStep step={1}>previousState</CodeStep> of the Action
- The <CodeStep step={2}>arguments</CodeStep> the action is called with, in this case an onChange `event`.

The Action receives the previous state because it works like a reducer. Every time the event fires, React ensures that the state updates are applied in the correct order. Like a reducer, sometimes, you may need to use the previous state to compute the new state (see TODO).

<Recipes titleText="The difference between useActionState and async transitions">

#### Updating the quantity with `useActionState` {/*updating-the-users-name-with-useactionstate*/}

This example adds `useActionState` to the example from [using async transitions to create an Action](#updating-the-quantity-in-an-action) to show the difference between creating actions with `useActionState` and creating Actions with async transitions directly. 

In this example, the `updateQuantity` function simulates a request to the server to update the item's quantity in the cart. This function *artificially returns the every other request after the previous* to simulate race conditions for network requests.

Try updating the quantity once, then update it quickly multiple times. Notice that the total always updates in order:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useState, useActionState } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, action, isPending] = useActionState(async (previousState, event) => {
    const savedQuantity = await updateQuantity(event.target.value);
    return savedQuantity;
  }, 1);
  
  return (
    <div>
      <h1>Checkout</h1>
      <Item action={action}/>
      <hr />
      <Total quantity={quantity} isPending={isPending} />
    </div>
  );
}

```

```js src/Item.js
export default function Item({action}) {
  return (
    <div className="item">
      <span>Eras Tour Tickets</span>
      <label htmlFor="name">Quantity: </label>
      <input
        type="number"
        onChange={action}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({ quantity, isPending }) {
  return (
    <div className="total">
      <span>Total:</span>
      <div>
        {isPending
          ? "🌀 Updating..."
          : `${intl.format(quantity * 9999)}`}
      </div>
    </div>
  );
}
```

```js src/api.js
let firstRequest = true;
export async function updateQuantity(newName) {
  return new Promise((resolve, reject) => {
    if (firstRequest === true) {
      firstRequest = false;
      setTimeout(() => {
        firstRequest = true;
        resolve(newName);
        // Simulate every other request being slower
      }, 1000);
    } else {
      setTimeout(() => {
        resolve(newName);
      }, 50);
    }
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}

.error {
  color: red;
}
```

</Sandpack>

This works because React handles actions from `useActionState` in order, waiting for the last request to finish before the next request. If this is not desirable, you can can add support for manually with an [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) or use async transitions directly (see: [Troubleshooting](TODO)).

For common cases, `useActionState` provides a simple API to create Actions that handle pending states and return values. For more complex cases, you can use async transitions directly or a library that supports Actions.

<Solution />

#### Creating an Action without `useActionState` {/*creating-an-action-without-useactionstate*/}

This example shows [the limitation of async transitions](/reference/react/useTransition#my-state-updates-in-async-transitions-are-out-of-order) which does not ensure correct ordering of updates.

In this example, the `updateQuantity` function simulates a request to the server to update the item's quantity in the cart. This function *artificially returns every other request after the previous* to simulate race conditions for network requests.

Try updating the quantity once, then update it quickly multiple times. You might see the incorrect total:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useState, useTransition } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, setQuantity] = useState(1);
  const [isPending, startTransition] = useTransition();
  // Store the actual quantity in separate state to show the mismatch.
  const [clientQuantity, setClientQuantity] = useState(1);
  
  const updateQuantityAction = event => {
    const newQuantity = event.target.value;
    setClientQuantity(newQuantity);
    // Update the quantity in an async transition.
    startTransition(async () => {
      const savedQuantity = await updateQuantity(newQuantity);
      startTransition(() => {
        setQuantity(savedQuantity);
      });
    });
  };

  return (
    <div>
      <h1>Checkout</h1>
      <Item action={updateQuantityAction}/>
      <hr />
      <Total clientQuantity={clientQuantity} savedQuantity={quantity} isPending={isPending} />
    </div>
  );
}

```

```js src/Item.js
export default function Item({action}) {
  return (
    <div className="item">
      <span>Eras Tour Tickets</span>
      <label htmlFor="name">Quantity: </label>
      <input
        type="number"
        onChange={action}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({ clientQuantity, savedQuantity, isPending }) {
  return (
    <div className="total">
      <span>Total:</span>
      <div>
        <div>
          {isPending
            ? "🌀 Updating..."
            : `${intl.format(savedQuantity * 9999)}`}
        </div>
        <div className="error">
          {!isPending &&
            clientQuantity !== savedQuantity &&
            `Wrong total, expected: ${intl.format(clientQuantity * 9999)}`}
        </div>
      </div>
    </div>
  );
}
```

```js src/api.js
let firstRequest = true;
export async function updateQuantity(newName) {
  return new Promise((resolve, reject) => {
    if (firstRequest === true) {
      firstRequest = false;
      setTimeout(() => {
        firstRequest = true;
        resolve(newName);
        // Simulate every other request being slower
      }, 1000);
    } else {
      setTimeout(() => {
        resolve(newName);
      }, 50);
    }
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}

.total div {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

.error {
  color: red;
}
```
</Sandpack>

<Solution />

</Recipes>


### Accessing latest state from an Action {/*accessing-latest-state-from-an-action*/}

Actions compose, so you can also call an Action from `useActionState` to access the last returned value and the pending state of the Action.

For example, if you wanted to access the state of an Action in a Child to show a loading spinner, you can wrap the Action in `useActionState`. Try updating the quantity, and notice the both spinners update in sync:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useState, useTransition } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, setQuantity] = useState(1);
  const [isPending, startTransition] = useTransition();

  const updateQuantityAction = event => {
    const newQuantity = event.target.value;
    // Update the quantity in an async transition.
    startTransition(async () => {
      const savedQuantity = await updateQuantity(newQuantity);
      startTransition(() => {
        setQuantity(savedQuantity);
      });
    });
  };

  return (
    <div>
      <h1>Checkout</h1>
      <Item action={updateQuantityAction}/>
      <hr />
      <Total quantity={quantity} isPending={isPending} />
    </div>
  );
}
```

```js src/Item.js active
import { useActionState } from "react";

export default function Item({ action }) {
  // Wrap the action in useActionState to access the latest state and pending state.
  const [quantity, updateAction, isPending] = useActionState(
    async (previousState, event) => {
      action(event);
      return event.target.value;
    },
    1
  );

  return (
    <div className="item">
      <span>Eras Tour Tickets x {isPending ? "🌀" : quantity}</span>
      <label htmlFor="name">Quantity: </label>
      <input type="number" onChange={updateAction} defaultValue={1} min={1} />
    </div>
  );
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({quantity, isPending}) {
  return (
    <div className="total">
      <span>Total:</span>
      <span>
        {isPending ? "🌀 Updating..." : `${intl.format(quantity * 9999)}`}
      </span>
    </div>
  )
}
```

```js src/api.js
export async function updateQuantity(newQuantity) {
  return new Promise((resolve, reject) => {
    // Simulate a slow network request.
    setTimeout(() => {
      resolve(newQuantity);
    }, 2000);
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}
```

</Sandpack>



### Using information returned by a form action {/*using-information-returned-by-a-form-action*/}

Call `useActionState` at the top level of your component to access the return value of an action from the last time a form was submitted.

```js [[1, 5, "state"], [2, 5, "formAction"], [3, 5, "action"], [4, 5, "null"], [2, 8, "formAction"]]
import { useActionState } from 'react';
import { action } from './actions.js';

function MyComponent() {
  const [state, formAction] = useActionState(action, null);
  // ...
  return (
    <form action={formAction}>
      {/* ... */}
    </form>
  );
}
```

`useActionState` returns an array with exactly two items:

1. The <CodeStep step={1}>current state</CodeStep> of the form, which is initially set to the <CodeStep step={4}>initial state</CodeStep> you provided, and after the form is submitted is set to the return value of the <CodeStep step={3}>action</CodeStep> you provided.
2. A <CodeStep step={2}>new action</CodeStep> that you pass to `<form>` as its `action` prop.

When the form is submitted, the <CodeStep step={3}>action</CodeStep> function that you provided will be called. Its return value will become the new <CodeStep step={1}>current state</CodeStep> of the form.

The <CodeStep step={3}>action</CodeStep> that you provide will also receive a new first argument, namely the <CodeStep step={1}>current state</CodeStep> of the form. The first time the form is submitted, this will be the <CodeStep step={4}>initial state</CodeStep> you provided, while with subsequent submissions, it will be the return value from the last time the action was called. The rest of the arguments are the same as if `useActionState` had not been used.

```js [[3, 1, "action"], [1, 1, "currentState"]]
function action(currentState, formData) {
  // ...
  return 'next state';
}
```

<Recipes titleText="Display information after submitting a form" titleId="display-information-after-submitting-a-form">

#### Display form errors {/*display-form-errors*/}

To display messages such as an error message or toast that's returned by a Server Action, wrap the action in a call to `useActionState`.

<Sandpack>

```js src/App.js
import { useActionState, useState } from "react";
import { addToCart } from "./actions.js";

function AddToCartForm({itemID, itemTitle}) {
  const [message, formAction] = useActionState(addToCart, null);
  return (
    <form action={formAction}>
      <h2>{itemTitle}</h2>
      <input type="hidden" name="itemID" value={itemID} />
      <button type="submit">Add to Cart</button>
      {message}
    </form>
  );
}

export default function App() {
  return (
    <>
      <AddToCartForm itemID="1" itemTitle="JavaScript: The Definitive Guide" />
      <AddToCartForm itemID="2" itemTitle="JavaScript: The Good Parts" />
    </>
  )
}
```

```js src/actions.js
"use server";

export async function addToCart(prevState, queryData) {
  const itemID = queryData.get('itemID');
  if (itemID === "1") {
    return "Added to cart";
  } else {
    return "Couldn't add to cart: the item is sold out.";
  }
}
```

```css src/styles.css hidden
form {
  border: solid 1px black;
  margin-bottom: 24px;
  padding: 12px
}

form button {
  margin-right: 12px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "^5.0.0"
  },
  "main": "/index.js",
  "devDependencies": {}
}
```
</Sandpack>

<Solution />

#### Display structured information after submitting a form {/*display-structured-information-after-submitting-a-form*/}

The return value from a Server Action can be any serializable value. For example, it could be an object that includes a boolean indicating whether the action was successful, an error message, or updated information.

<Sandpack>

```js src/App.js
import { useActionState, useState } from "react";
import { addToCart } from "./actions.js";

function AddToCartForm({itemID, itemTitle}) {
  const [formState, formAction] = useActionState(addToCart, {});
  return (
    <form action={formAction}>
      <h2>{itemTitle}</h2>
      <input type="hidden" name="itemID" value={itemID} />
      <button type="submit">Add to Cart</button>
      {formState?.success &&
        <div className="toast">
          Added to cart! Your cart now has {formState.cartSize} items.
        </div>
      }
      {formState?.success === false &&
        <div className="error">
          Failed to add to cart: {formState.message}
        </div>
      }
    </form>
  );
}

export default function App() {
  return (
    <>
      <AddToCartForm itemID="1" itemTitle="JavaScript: The Definitive Guide" />
      <AddToCartForm itemID="2" itemTitle="JavaScript: The Good Parts" />
    </>
  )
}
```

```js src/actions.js
"use server";

export async function addToCart(prevState, queryData) {
  const itemID = queryData.get('itemID');
  if (itemID === "1") {
    return {
      success: true,
      cartSize: 12,
    };
  } else {
    return {
      success: false,
      message: "The item is sold out.",
    };
  }
}
```

```css src/styles.css hidden
form {
  border: solid 1px black;
  margin-bottom: 24px;
  padding: 12px
}

form button {
  margin-right: 12px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "^5.0.0"
  },
  "main": "/index.js",
  "devDependencies": {}
}
```
</Sandpack>

<Solution />

</Recipes>

## Troubleshooting {/*troubleshooting*/}

### My action can no longer read the submitted form data {/*my-action-can-no-longer-read-the-submitted-form-data*/}

When you wrap an action with `useActionState`, it gets an extra argument *as its first argument*. The submitted form data is therefore its *second* argument instead of its first as it would usually be. The new first argument that gets added is the current state of the form.

```js
function action(currentState, formData) {
  // ...
}
```

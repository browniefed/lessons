## Intro

Undo logic is one of the more difficult things to implement in any application. State management is hard enough and then you add in management of state over time making it even more difficult. We'll dive into how we can use the [Immer](https://github.com/mweststrate/immer) library to lend us a hand in managing our state and making undo easy.

![](https://images.codedaily.io/lessons/general/immerUndo/rendered_with_items.png)

## Setup Scenario

This technique will work with anything, and any data structure. We'll focus on a list of names that are stored in an object form.

```js
import React, { Component } from "react";
import "./app.css";

class App extends Component {
  state = { value: "", items: [] };
  render() {
    return (
      <div className="App">
        {this.state.items.map(({ name }) => {
          return <div>{name}</div>;
        })}
        <form>
          <input
            autoFocus
            value={this.state.value}
            onChange={e => this.setState({ value: e.target.value })}
          />
          <button type="submit">Add</button>
        </form>
      </div>
    );
  }
}

export default App;
```

![](https://images.codedaily.io/lessons/general/immerUndo/rendered.png)

## Add Immer for Adding

Now lets use Immer with `setState` for adding name to the items array. We'll need to bring in `produce` from Immer which takes a state, and produces the next state in an immutable fashion.

![](https://images.codedaily.io/lessons/general/immerUndo/rendered_with_items.png)

```js
import produce, { applyPatches } from "immer";
```

Our `handleAdd` function will be from a `<form>` submission so we'll need to `preventDefault`. Our `produce` function takes ours our `this.state` and the second argument is a function to modify. We'll call it `draft` as it's a draft for the next state.

Our `draft` can be mutated but we'll still produce an immutable return. We then do a `setState` to update our state.
The third argument is handling our patches for what data was modified.

```js
handleAdd = e => {
  e.preventDefault();

  const nextState = produce(
    this.state,
    draft => {
      draft.value = "";
      draft.items.push({ name: this.state.value });
    },
    this.handleAddPatch
  );
  this.setState(nextState);
};
```

Immer produces two patches. The first is `patch` is the patch of what we did. In our case we changed `name` to an empty string. Then we pushed an item onto an array.

The inverse patch which we care about is the opposite of what we did.

```js
undo = [];
state = { value: "", items: [] };

handleAddPatch = (patch, inversePatches) => {
  this.undo.push(inversePatches);
};
```

Now we wire up our form so when you press the submit button, or press enter it will submit our form.

```js
<form onSubmit={this.handleAdd}>
  <input
    autoFocus
    value={this.state.value}
    onChange={e => this.setState({ value: e.target.value })}
  />
  <button type="submit">Add</button>
</form>
```

## What is a Patch

Immer will produce a patch that closely follows [RFC-6902 JSON patch standard](https://tools.ietf.org/html/rfc6902) except it's `path` value isn't a string but an array so it is easier to handle the patches. It's a JSON structure that's a command for how to modify a JSON object.

Instead of reading a standard we can just look at what the patches are to get an understanding.

This is the `patch` for adding a new item. The `patch` variable we received above is an array of changes. We made 2 changes so we have 2 patches and inverse patches.

Our patch here has an `add` operation. We added one new `value` to the `path` `items.0` so we added it to the first index. We also modified the `value` on our state object to an empty string which is a `replace` operation.

```js
[
  {
    "op": "add",
    "path": ["items", 0],
    "value": {
      "name": "Test1"
    }
  },
  {
    "op": "replace",
    "path": ["value"],
    "value": ""
  }
]
```

If we take a look at the inverse patch this is storing an operation that when applied to an object that has the previous `patch` applied would result in the original outcome. In the end an undo stack is a small series of modifications over a period of time. So tracking these small changes we can apply them and achieve our previous states.

So you can see here we started with an empty array so the command to achieve the original state would be a `replace` operation on the length which would clear out the array back to empty.

We also started with `Test1` as our submitted value so we also have a `replace` operation to replace the value.

```js
[
  {
    "op": "replace",
    "path": ["items", "length"],
    "value": 0
  },
  {
    "op": "replace",
    "path": ["value"],
    "value": "Test1"
  }
]
```

## Add Undo

Now that we have an understanding of what are patches are lets implement the undo feature.
Immer provides a function called `applyPatches` that will take an array of patches, execute them against an object and return the new state.

We pop an undo patch off of our `this.undo` array. If we have one then we call our `applyPatches` function with our state, and our patches and do a `setState`.

![](https://images.codedaily.io/lessons/general/immerUndo/add_undo.gif)

```js
handleUndo = () => {
  const undo = this.undo.pop();
  if (!undo) return;
  this.setState(applyPatches(this.state, undo));
};
```

Now we add a button so you can do the undo action.

```js
<form onSubmit={this.handleAdd}>
  <input
    autoFocus
    value={this.state.value}
    onChange={e => this.setState({ value: e.target.value })}
  />
  <button type="submit">Add</button>

  <div className="actions">
    <button onClick={this.handleUndo} type="button">
      Undo
    </button>
  </div>
</form>
```

## Add Clear

Adding a clear button can also be implemented and made undoable. This is just like our `handleAdd` but instead of pushing an item onto the array we just clear out the values. Here we can assign an empty array to our items, or additionally you could set the `length` of `draft.items` to 0 like so `drag.items.length = 0`. Both will clear the values but will also produce different patches. Not something you need to worry about since the effects are the same in this case.

![](https://images.codedaily.io/lessons/general/immerUndo/clear_undo.gif)

```js
handleClear = () => {
  const nextState = produce(
    this.state,
    draft => {
      draft.items = [];
      draft.value = "";
    },
    this.handleAddPatch
  );
  this.setState(nextState);
};
```

Now we wire up our button so you can clear it.

```js
<form onSubmit={this.handleAdd}>
  <input
    autoFocus
    value={this.state.value}
    onChange={e => this.setState({ value: e.target.value })}
  />
  <button type="submit">Add</button>

  <div className="actions">
    <button onClick={this.handleUndo} type="button">
      Undo
    </button>
    <button onClick={this.handleClear} type="button">
      Clear
    </button>
  </div>
</form>
```

## Ending

Not only can Immer do undo stacks but you could also implement redo as well. You will just need to fully understand exactly what's happening with your patches and understand the current state your data is in so that the patches apply correctly. Applying a patch may invalidate some of your other patches.

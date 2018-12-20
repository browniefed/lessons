## Intro

One common interaction with dropdowns is closing when anything else on the page is clicked. Within React though with controlled dropdowns that are shown by updating state you need to add additional code in order to properly setup this interaction.

We also need to be sure and close the menu when the button is clicked rather than re-opening it.

## Adding a Button

First we need to add a button and a container to hold onto our menu. This is what our markup will look like, and we'll just use the html entity of 3 lines to indicate that we have menu.

```js
render() {
    return (
      <div className="App">
        <div className="container">
          <button type="button" class="button">
            ☰
          </button>
        </div>
      </div>
    );
  }
```

This container being `inline-block` and `relative` is very important. Our menu will need to be absolutely positioned and it will be positioned in accordance to the first parent it hits with `position: relative` set.

So if we don't make our container relative it would be positioned relative to the body which we dont' want.

Additionally giving the container `display: inline-block` means the width/height will adjust automatically to however large the button is. If it were `block` then the width of the container would be `100%` and if you pressed to the right of the button it wouldn't trigger our future "click outside" code and the menu wouldn't close.

```
.container {
  position: relative;
  display: inline-block;
}
.button {
  padding: 0;
  width: 50px;
  border: 0;
  background-color: #fff;
  color: #333;
  cursor: pointer;
  outline: 0;
  font-size: 40px;
}
```

The button CSS isn't too important, but because we're using html entities to control the size of the menu icon we need to use `font-size`. Additionally we kill some default padding/border, and just give the button a defined width.

![](https://images.codedaily.io/lessons/general/click_outside/button.png)

## Add the Dropdown

Now we need to add our dropdown. This is just another div, with a un-ordered list inside. We need to render this dropdown as a child of our `container` which we set to `position: relative`.

```js
<div className="container">
  <button type="button" class="button">
    ☰
  </button>
  <div class="dropdown">
    <ul>
      <li>Option 1</li>
      <li>Option 2</li>
      <li>Option 3</li>
      <li>Option 4</li>
    </ul>
  </div>
</div>
```

The key bit of CSS here is that we `position: absolute` as well as set the `top: 100%`. This will allow the menu to not care how big the button is but to move the dropdown to start at the very bottom of relative container. 

Our relative container is `inline-block` so if it grows/shrinks the menu will always be hanging off the bottom of it.

We also style our dropdown list a little. Generally you would set global styling for the `ul/li` but this CSS is just for demo sake.

```
.dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  width: 300px;
  z-index: 2;
  border: 1px solid rgba(0, 0, 0, 0.04);
  box-shadow: 0 16px 24px 2px rgba(0, 0, 0, 0.14);
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
li {
  padding: 8px 12px;
}

li:hover {
  background-color: rgba(0, 0, 0, 0.14);
  cursor: pointer;
}
```

![](https://images.codedaily.io/lessons/general/click_outside/menu.png)

## Open/Close Menu

Our menu is now just sitting open so lets allow for our button to actually do something. We need to attach a click listener to the button so we can trigger our action. The action will be to adjust our state to toggle an `open` variable.

We use the `setState` callback method here because we are referencing previous state and want to account for the future async world. This style will preserve the order of which `setState` is called so updates to state are made in the correct order.

```js
class App extends Component {
  state = {
    open: false,
  };
  handleButtonClick = () => {
    this.setState(state => {
      return {
        open: !state.open,
      };
    });
  };
}
```

Now that we have state toggling our `open` we can then decide whether or not to render our dropdown or not. 

```js
<div className="container">
  <button type="button" class="button" onClick={this.handleButtonClick}>
    ☰
  </button>
  {this.state.open && (
    <div class="dropdown">
      <ul>
        <li>Option 1</li>
        <li>Option 2</li>
        <li>Option 3</li>
        <li>Option 4</li>
      </ul>
    </div>
  )}
</div>
```

## The Problem

Now we have a problem. If the user clicks outside of the dropdown it will stay open. In order to close the menu they'd have to go click on the menu button again.

The technique here is that we need to register a click on the document, and when a user clicks anywhere we check if the click occurred in our container.

If it didn't occur in our container then we can close the menu, or do some other action that you desire.


## Add a Ref

We will need the raw DOM element here so we will need to use `refs`. React has a method called `createRef`. 

```js
container = React.createRef();
state = {
  open: false,
};
```

We then pass our ref to the `ref` property on our DOM element and we will then have access to this container div later.

```js
<div className="container" ref={this.container}>
```

## Clicking Outside

We wire up click listeners on the document for `mousedown`. Then remove in our `componentWillUnmount` to properly cleanup our listeners.

If you wanted to wait until `mouseup` before closing that would also be an option.

```js
componentDidMount() {
    document.addEventListener("mousedown", this.handleClickOutside);
}
componentWillUnmount() {
  document.removeEventListener("mousedown", this.handleClickOutside);
}
```

We need to check to make sure that our `current` is actually filled in with a DOM element. Then using the DOM method `contains` we ask our container if we have the `event.target` which is the DOM element that was clicked.

```js
handleClickOutside = event => {
  if (this.container.current && !this.container.current.contains(event.target)) {
    this.setState({
      open: false,
    });
  }
};
```

If we don't have the clicked target then that means it's outside of our `container` and we need to close our menu. So we call `setState` and set `open` to false.

Our `button` is inside of our container so it will run the toggle code as normal and close the menu when it's clicked.

![](https://images.codedaily.io/lessons/general/click_outside/menuopen.gif)

## Ending

Overall this is a very common pattern and can be generalized inside of any component that is needed or even in the future can be generalized with a `hook`.
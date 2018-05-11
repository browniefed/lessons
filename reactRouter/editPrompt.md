## Intro

Data loss prevention is a UX feature that should be present on all web apps that have longer form user input data. There are many times when a user has spent minutes or even hours entering data and then accidentally clicks a link without saving or didn't even know things were still unsaved. The user should be notified via a prompt and the data should have a chance to be saved.

React Router has a built in mechanism that allows for just that.

![](https://images.codedaily.io/lessons/reactRouter/editPrompt.gif)

## Setup

First we're going to do some route setup. We just create 2 links, and 2 routes. We have a `Home` route that won't have any user input, and we'll have a `Profile` route with user input.

```js
class App extends Component {
  render() {
    return (
      <div>
        <div className="links">
          <Link to="/" className="link">
            Home
          </Link>
          <Link to="/profile" className="link">
            Profile
          </Link>
        </div>
        <div className="tabs">
          <Switch>
            <Route path="/" exact component={Home} />
            <Route path="/profile" component={Profile} />
          </Switch>
        </div>
      </div>
    );
  }
}
```

## Add Interactivity

In our `Profile` route we will import the `Prompt` module from React Router. We have some `state` to hold onto the name of the user, and added an input to allow for the user to enter their name.

```js
import React, { Component } from "react";

class Profile extends Component {
  state = {
    name: "",
  };
  render() {
    return (
      <div>
        <div>
          <div>Nice looking profile! What's your name?</div>
          <input value={this.state.name} onChange={e => this.setState({ name: e.target.value })} />
        </div>
      </div>
    );
  }
}

export default Profile;
```

## Add Prompt

Now comes in our `Prompt`. This component doesn't take any children, you render it and control it with a few props. The first prop is the `when` component. This is what enables the prompt. When it's `true` it'll show the prompt message or if it's `false` will let any navigation happen.

Then we have our `message`, this message can either be a string or a function. The function method will receive the next location to navigate to.

There is a feature of this function method on `Prompt` that is very useful. When receiving the next location, that location may be a nested route, or imagine if you had a wizard form and just the section of the form was changing but the component where you held your form state would still be mounted.

If this is the case and you detect that with the next location given to this function, then you can return `true`.

```js
render() {
    return (
      <div>
        <div>
          <Prompt
            when={!!this.state.name}
            message={location => `Are you sure you want to go to ${location.pathname}`}
          />
          <div>Nice looking profile! What's your name?</div>
          <input value={this.state.name} onChange={e => this.setState({ name: e.target.value })} />
        </div>
      </div>
    );
  }
```

In our case we don't have that sort of setup but it's something to be aware. We will prompt the user if they'd like to navigate away whenever the `name` field on state has any data in it, and upon clicking yes will navigate, or no will stay in the current location. 

## Ending

The `Prompt` component from React Router makes it easier to achieve the appropriate user experience when it comes to implementing large form data entry.

![](https://images.codedaily.io/lessons/reactRouter/editPrompt.gif)

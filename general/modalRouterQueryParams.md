## Introduction

There are times where creating a full page doesn't make sense for a particular route in your application. Generally this means creating a Modal. However in the event that a user wants to link to it we need it to exist as a route.

An example we'll focus on is a login modal. We want to link to the modal, however it should be able to be linked and appear over any page dynamically. Depending on your route structure this can be tricky, so we'll use a query param. So let's explore how to setup a query param modal that can be rendered over any page.

## Setup Routes

First we need to setup our pages. Our routes are just 2 pages, the home page and the profile page. These could be any number of pages, but these will sit inside the `Switch`. Whenever the paths are matched they will render. 

```js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";

import { BrowserRouter, Switch, Route } from "react-router-dom";
import HomePage from "./pages/home";
import ProfilePage from "./pages/profile";

const routes = (
  <BrowserRouter>
    <Switch>
      <Route exact path="/" component={HomePage} />
      <Route path="/profile" component={ProfilePage} />
    </Switch>
  </BrowserRouter>
);

ReactDOM.render(routes, document.getElementById("root"));
```

## Pages

Now here are our 2 pages, they both have links to the `Login` link. We haven't wired these up just yet but we'll do that later.

```js
import React, { Component } from "react";
import { Link } from "react-router-dom";

export default class HomePage extends Component {
  render() {
    return (
      <div>
        <Link to="/profile">Go To Profile</Link>
        <Link>Login</Link>
      </div>
    );
  }
}
```

```js
import React, { Component } from "react";
import { Route, Link } from "react-router-dom";

export default class ProfilePage extends Component {
  render() {
    return (
      <div>
        <Link>Login</Link>
      </div>
    );
  }
}
```

## Create a Modal

Rather than focusing on a specific Modal we'll create a reusable one. We'll be using `createPortal` from `react-dom`. We don't want to render to `document.body` so in our `index.html` we'll add another div to render to which we'll give an id of `modal_root`.


```html
<div id="root"></div>
<div id="modal_root"></div>
```

`createPortal` will render React provided, to a different place in the DOM and ensure that all appropriate context is passed along as well. Think of it as a "transport this HTML somewhere else". This allows for you to render something that makes sense hierarchically but functionally needs to exist somewhere else in the DOM.

The wrapping `div` will apply our styling, so it will be a fixed `div` covering the screen with a dark background. Upon click it will call a passed in `onClick` method which later we'll use for closing the modal.

We pass in our `children` which means any React that we want will be rendered in our modal, and transported to `modal_root`.

```js
import React, { Component } from "react";
import { createPortal } from "react-dom";

const modalStyle = {
  position: "fixed",
  left: 0,
  top: 0,
  bottom: 0,
  right: 0,
  backgroundColor: "rgba(0,0,0,.2)",
  color: "##FFF",
  fontSize: "40px",
};
export default class Modal extends Component {
  render() {
    return createPortal(
      <div style={modalStyle} onClick={this.props.onClick}>
        {this.props.children}
      </div>,
      document.getElementById("modal_root"),
    );
  }
}
```

## Create Login Modal

Now we will create our login modal route. If we're rendering this as a page and we want it to work anywhere that we render we can't hard code any specific paths, they must be dynamic.

The behavior we desire is that if the background is clicked that the modal closes. We can do this with the `history` prop passed in by React Router.

Additionally rather than referencing `match.url` which is the path is matched by React Router. So instead we use the `location.pathname` which will return the current path in URL minus the query params.

```js
import React, { Component } from "react";
import Modal from "./modal";

export default class LoginPage extends Component {
  render() {
    return (
      <Modal
        onClick={() => {
          this.props.history.push(this.props.location.pathname);
        }}
      >
        <div
          style={{
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            height: "100%",
          }}
        >
          Login modal
        </div>
      </Modal>
    );
  }
}
```

## Render Routes Base on Query Params

To make this work we need to render our route outside of the `Switch`. The reason this will work is because the Route will render all the time. We've supplied it a path of `/` and did not put the `exact` prop on it. So anytime the route changes in our application this route will re-render.

This technique of depending on a query param is great for when you have a lot of `exact` routes. It also would not make sense to have the path be a hard coded `/profile/login`. So depending on a query param is an easy solution.

```js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";

import { BrowserRouter, Switch, Route } from "react-router-dom";
import HomePage from "./pages/home";
import ProfilePage from "./pages/profile";
import Login from "./pages/login";

const routes = (
  <BrowserRouter>
    <Switch>
      <Route exact path="/" component={HomePage} />
      <Route path="/profile" component={ProfilePage} />
    </Switch>
    <Route path="/" component={Login} />
  </BrowserRouter>
);

ReactDOM.render(routes, document.getElementById("root"));
```

## Login Modal Route Based on Query

Now that we're rendering our login page, lets make the `LoginPage` modal actually respect the query param. We use the `URLSearchParams`, which will take the query params, also called the `search` portion of the URL, and provide us with a bunch of helpers.

We grab our params, and do a `.get("login")` to check the existence of the login query param. If it exists we return our login modal, otherwise nothing will return and nothing will render.

This is supported by all browsers except for IE. You do not need to use `URLSearchParams` you can parse and access the query params however you see fit.

```js
import React, { Component } from "react";
import Modal from "./modal";

export default class LoginPage extends Component {
  render() {
    let params = new URLSearchParams(this.props.location.search);

    return (
      params.get("login") && (
        <Modal
          onClick={() => {
            this.props.history.push(this.props.location.pathname);
          }}
        >
          <div
            style={{
              display: "flex",
              alignItems: "center",
              justifyContent: "center",
              height: "100%",
            }}
          >
            Login modal
          </div>
        </Modal>
      )
    );
  }
}
```

## Fix Links

We need to fix our links, but because we are rendering it on any page we should craft dynamic links. The `Link` component from React Router can receive a string but also an object for its `to` prop.

We supply an object and provide a `pathname` which will be our `match.url` so the path that we are matching on, and then additionally a `search` key and pass in `?login=true` so that it's not undefined and will register accordingly with our `URLSearchParams`. 

```js
<Link to={{ pathname: this.props.match.url, search: "?login=true" }}>Login</Link>
```

## Ending

Now we have a dynamic modal route based upon query params. The main technique to take away here is that you can render Routes anywhere. They don't have to be just in a top level `Switch`. They will render when the `path` is matched, and that's what we take advantage of here. We can create always rendering routes that will allow us to create these types of interfaces.

You can check out the code here [https://github.com/codedailyio/ReactRouterModal](https://github.com/codedailyio/ReactRouterModal) and or check out the [live demo](https://codesandbox.io/s/github/codedailyio/ReactRouterModal).

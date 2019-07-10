# Introduction

Modals present a tricky situation for many applications. Are they a route? Are they just presenting info? Do users need to get back to whatever they're presenting?

Regardless of the situation in React most modals are presented by toggling a piece of state. However on a refresh that state will no longer exist so the modal will close. The only mechanism for storing state that is shareable across users is via the URL. Lets explore how to create a modal, and how we can go about turning into a route while not un-rendering the current route.

# Setup Routes

So we need to setup our app routes. We'll go with a 2 page app. The first is a home route, the second is a route to the Profile page. The `Switch` route will render the first route that is matched. This is all a very generic setup. One thing to point out is that there is no `exact` flag set on our `/profile` route.

So that means our profile route will match anytime you hit `/profile` or anything beyond that like `/profile/edit`.

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

# Pages

The 2 pages above are React components with links.

```js
import React, { Component } from "react";
import { Link } from "react-router-dom";

export default class HomePage extends Component {
  render() {
    return (
      <div>
        <Link to="/profile">Go To Profile</Link>
      </div>
    );
  }
}
```

Here we have a link to the Edit Profile "page" that will be our modal. This isn't a valid link just yet as there is no `to` prop, but we'll get there. 

```js
import React, { Component } from "react";
import { Route, Link } from "react-router-dom";

export default class ProfilePage extends Component {
  render() {
    return (
      <div>
        <Link>Edit Profile</Link>
      </div>
    );
  }
}
```

# Create a Modal

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
  color: "#FFF",
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

# Render Dynamic Subroute

Now that we have our modal, pages, and routes all setup lets look at how to put it all together. Upon clicking an `Edit Profile` link you might want a modal to pop up that covers the existing page.

Being a modal route it doesn't make sense for a top level route, and because it's dealing with the profile it makes sense to colocate this route. This is the power of React Router and it's ability to render routes anywhere.

The concept is that a `Route` takes a `path`. If that Route is rendered and the path is matched then the supplied React will render. 

Generally routes are though of as full pages when in fact it could control a full page, or a small component. An example of this would be tabs. You may have multiple tabs on a page but want people to navigate to them. You can add in tab specific routes to that page. So one route to render the wrapping page, and any number of sub-tabs rendering with in that page. The URL will match to render the main outer page, and then the tab paths can be setup to render only when those URLs are visited.

Lets look at how this applies to our modal route. First lets fix our `Edit Profile` link.

Rather than specifying `/profile` we can dynamically link to whatever page we're currently matched on. The `match.url` is supplied by React Router and is the `path` from the `Route` that is being rendered.

So because we had just `/profile` our `match.url` will always be `/profile`.

```js
<Route path="/profile" component={ProfilePage} />
```

Event when we link to `/profile/edit` the `Route` that is rendering our `ProfilePage` only matched `/profile`. So our `match.url` will always be `/profile`.

So now our link can reference `/profile` dynamically, and if the top level route ever was changed our `Edit Profile` would still continue to work and we wouldn't need to search our entire codebase to fix the route.

```js
<Link to={`${this.props.match.url}/edit`}>Edit Profile</Link>
```


Now that we're on the `ProfilePage` we can render a new `Edit Profile` route, and just like our `Link` above we make it dynamic. We pass in our `match.url` along with the added `/edit`. When the user clicks or visits `/profile/edit` our `Route` will render the `Modal`.

```js
import React, { Component } from "react";
import { Route, Link } from "react-router-dom";
import Modal from "./modal";

export default class ProfilePage extends Component {
  render() {
    return (
      <div>
        <Link to={`${this.props.match.url}/edit`}>Edit Profile</Link>

        <Route
          path={`${this.props.match.url}/edit`}
          render={() => {
            return (
              <Modal
                onClick={() => {
                  this.props.history.push(this.props.match.url);
                }}
              >
                <div
                  style={{
                    display: "flex",
                    alignItems: "center",
                    justifyContent: "center",
                    height: '100%'
                  }}
                >
                  Edit Profile Modal!
                </div>
              </Modal>
            );
          }}
        />
      </div>
    );
  }
}
```

Finally we add in an `onClick` to our Modal so when it's clicked we dismiss and `push` to the current `match.url`. Because we are in the `ProfilePage` and reference `this.props` of the profile page that `match.url` will reference `/profile`.

If you were to grab the props passed to the `render={(props) => {}}` function of route the `props` from that would the the `props.match.url` equal to `/profile/edit`.

# End

I want to be clear, this works because the original `/profile` route that we rendered did not have an `exact` prop. If we added an `exact` prop this strategy would not work. It's important to think about how you design your components and routes.

If you needed to render other routes at the path `/profile` you could have the top level `/profile` not be exact like we have here, and then supply other more specific routes as subroutes inside of your `ProfilePage`. You can add in all the necessary design, modal edit paths, and then other subroutes would render totally different components as necessary.

You can check out the code here [https://github.com/codedailyio/ReactRouterModal](https://github.com/codedailyio/ReactRouterModal) and or check out the [live demo](https://codesandbox.io/s/github/codedailyio/ReactRouterModal).

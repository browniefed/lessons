# Introduction

Modals present a tricky situation for many applications. Are they a route? Are they just presenting info? Do users need to get back to whatever they're presenting?

Regardless of the situation in React most modals are presented by toggling a piece of state. However on a refresh that state will no longer exist so the modal will close. The only mechanism for storing state that is shareable across users is via the URL. Lets explore how to create a modal, and how we can go about turning into a route while not un-rendering the current route.

# Setup Routes

So we need to setup our app routes. We'll go with a 2 page app. The first is a home route, the second is a route to the Profile page. The `Switch` route will render the first route that is matched. This is all a very generic setup. One thing to point out is that there is no `exact` flag set on our `/profile` route.

So that means our profile route will match anytime you hit `/profile` or anything beyond that `/profile/edit`.

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

Rather than focusing on a specific Modal
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

```js
import React, { Component } from "react";
import { Route, Link } from "react-router-dom";
import Modal from "./modal";

export default class ProfilePage extends Component {
  render() {
    return (
      <div>
        <Link to={`${this.props.match.url}/modal`}>Edit Profile</Link>
        <Link to={{ pathname: this.props.match.url, search: "?login=true" }}>Login</Link>

        <Route
          path={`${this.props.match.url}/modal`}
          render={() => {
            return (
              <Modal
                onClick={() => {
                  this.props.history.push("/profile");
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
                  Look at the modal!
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

# End

https://github.com/codedailyio/ReactRouterModal
https://codesandbox.io/s/github/codedailyio/ReactRouterModal

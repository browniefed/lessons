# Introduction

# Setup Routes

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
  </BrowserRouter>
);

ReactDOM.render(routes, document.getElementById("root"));
```

# Pages

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
        <Link>Edit Profile</Link>
        <Link>Login</Link>
      </div>
    );
  }
}
```

# Create a Modal

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

# Create Login Modal

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

# Render Routes Base on Query Params

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

# Fix Links

```js
<Link to={{ pathname: this.props.match.url, search: "?login=true" }}>Login</Link>
```

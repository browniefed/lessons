## Intro

A common pattern in most web applications is hiding unnecessary data behind tabs. However in defining these tabs you should make the data accessible in the event the user bookmarks, or shares the link. The method to share on the web is with a URL. Whether it's a absolute path, or a query param you should provide a mechanism to get back to the tab the user selected.

React Router supports this and allows for you to use normal React patterns to accomplish it.

![](https://images.codedaily.io/lessons/reactRouter/dynamicTabs.gif)

## Setup Router

First off we need to setup a router to render our app. We import our `BrowserRouter`, render a `Switch` to select between our routes.
We only really have a single route that we give a path `/home` to and our component is our `App`.

Don't worry too much about the `Redirect` it's merely so that when navigating to `/` it will redirect to our route. Essentially it's for demonstration purposes only.

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import { BrowserRouter, Route, Redirect, Switch } from "react-router-dom";

const routes = (
  <BrowserRouter>
    <Switch>
      <Route path="/home" component={App} />
      <Redirect from="/" to="/home" />
    </Switch>
  </BrowserRouter>
);

ReactDOM.render(routes, document.getElementById("root"));
```

## Create Tabs

Inside of our `App.js` file we need to create some tabs. These are going to be tab routes, they'll be the components that we render based upon the URL.

```js
const Profile = () => <div>You're on the Profile Tab</div>;
const Comments = () => <div>You're on the Comments Tab</div>;
const Contact = () => <div>You're on the Contact Tab</div>;
```

These components are just normal function React components.

## Create our Links

Now we need to setup links that will be our tab controls. We need to setup our `to` links to be absolute paths so that is why we need to add the `/home` to both `/comments` and `/contact`. The `/` is taken care of by the redirect up above.

```js
import { Link } from "react-router-dom";

class App extends Component {
  render() {
    return (
      <div>
        <h1>Hey welcome home!</h1>
        <div className="links">
          <Link to={`/`} className="link">Profile</Link>
          <Link to={`/home/comments`} className="link">Comments</Link>
          <Link to={`/home/contact`} className="link">Contact</Link>
        </div>
      </div>
    );
  }
}
```

## Setup Nested Tabs

Now we need to setup our tabs. React Router allows us to setup routes anywhere, so we don't need a static route config up top. 
We setup our route `path` to map to our `to` links. The `exact` prop on the first profile route means that whenever we want to match the `Profile` tab route we must be exactly at `/home`. Otherwise without the `exact` prop it won't render the other 2 tab routes.

```js
import { Switch, Route, Link } from "react-router-dom";

<div className="tabs">
  <Switch>
    <Route path={`/home`} exact component={Profile} />
    <Route path={`/home/comments`} component={Comments} />
    <Route path={`/home/contact`} component={Contact} />
  </Switch>
</div>
```

## Make Tab Routes Dynamic

One problem with this structure is that our routes need to be absolute paths. So if we wanted to change our base route from `/home` to something else we would need to change it everywhere on every link, ever `Route`, etc.

So rather than statically defining them we can dynamically configure our route paths based upon the `Route` that was matched from above. The `match` prop that is passed in will always be whatever we have defined here `<Route path="/home" component={App} />`.

```js
render() {
  const { path } = this.props.match;

  return (
    <div>
      <h1>Hey welcome home!</h1>
      <div className="links">
        <Link to={`${path}`} className="link">Profile</Link>
        <Link to={`${path}/comments`} className="link">Comments</Link>
        <Link to={`${path}/contact`} className="link">Contact</Link>
      </div>
      <div className="tabs">
        <Switch>
          <Route path={`${path}`} exact component={Profile} />
          <Route path={`${path}/comments`} component={Comments} />
          <Route path={`${path}/contact`} component={Contact} />
        </Switch>
      </div>
    </div>
  );
}
```

Here you can see our `to` on our `Link` is defined dynamically. Our `Route` tabs that we setup are all defined dynamically based upon our `match.path`. Then we can append the additional path that we want to match for each of our tabs.

Not only does it allow us to dynamically route but it makes our Tabs reusable. Imagine if we wanted to render tab routes with the same endings but the top would match differently. This allows for you to reuse the tabs and adjust automatically for the top level.

## Change Top Level

So with this setup it doesn't matter what we define at the top level. We can change it from `/home`, to something like `/crazyCoolNewRoute`. Something more practical like `/profile` would work to.

```js
const routes = (
  <BrowserRouter>
    <Switch>
      <Route path="/crazyCoolNewRoute" component={App} />
      <Redirect from="/" to="/crazyCoolNewRoute" />
    </Switch>
  </BrowserRouter>
);
```

## Ending

Overall this just scratches the surface of some of the power that React Router provides. We've defined dynamic tabs here, but all paths can be dynamic.

![](https://images.codedaily.io/lessons/reactRouter/dynamicTabs.gif)


https://codesandbox.io/s/github/browniefed/tmp/tree/reactRouterDynamicRoutes
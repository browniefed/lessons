## Intro

There may be a time in your application when you need to compare the previous route id to the current route id to load data or respond in some way. We can use the internal mechanisms of React Router to match any route we want.

![](https://images.codedaily.io/lessons/reactRouter/matchPath.gif)

## The Issue

If we take a look at this top level route, we can see that we want anything that matches `/profile` to pass on through to our `App` component. We could theoretically fix this by passing in `/profile/:profileId` and only rendering if we have an id match. But hypothetically speaking maybe you want to visit `/profile` and also match `/profile/:profileId` and pass through to the same component that is holding onto state.

```js
<BrowserRouter>
  <Route path="/profile" component={App} />
</BrowserRouter>
```

Well if we have this setup, and then visit `/profile/1` our `App` component `this.props.match` would look like this. It's only providing the `match` information for the path we specified on the route which is `/profile` without any ID.

```js
{
  "path": "/profile",
  "url": "/profile",
  "isExact": false,
  "params": {}
}
```

There are a few ways around this but we'll show a way to use the internal methods to make this work. This will also work outside of your React components, so if you have routes in Redux and want to utilize this method you can do just that.

## Create the Routes

First off we'll create our routes. We'll have a `SelectProfile` route that just instructs the person to select a link.
Then we have our `Profile` component. We will render the `profileId` you're viewing, but when we receive `loading` prop that is true we'll render a loading message.

```js
import React, { Component } from "react";
import { Route } from "react-router-dom";
import "./app.css";

const Profile = ({ match, loading }) => {
  if (loading) return <div>Loading...</div>;
  return <div>You're on the Profile {match.params.profileId}</div>;
};
const SelectProfile = () => <div>Select a Profile</div>;

class App extends Component {
  state = {
    loading: false,
  };
  render() {
    return <div />;
  }
}
```

## Setup Links

No we'll setup our links. We'll have our Home route setup to be `/profile` without an ID, and then we'll have 2 other links to navigate between 2 profiles with different IDs.

```js
import { Link } from "react-router-dom";
//...
class App extends Component {
  state = {
    loading: false,
  };
  render() {
    return (
      <div>
        <div className="links">
          <Link to="/profile" className="link">
            Home
          </Link>
          <Link to="/profile/1" className="link">
            Profile 1
          </Link>
          <Link to="/profile/2" className="link">
            Profile 2
          </Link>
        </div>
      </div>
    );
  }
}
```

## Setup Routes

Our route code below our links will use a `Switch` to only render the first item that it matches. Our main `Route` will be our `/profile` that we'll render without an ID, we must use the `exact` prop to tell it to only match it's `path` exactly, otherwise it would match for all routes that even have an ID.

Our second route will match a profile with a `profileId`. We must use the `render` prop here if we want to be able to pass down additional data like the `loading` state.

```js
import { Switch, Route, Link } from "react-router-dom";

//...

<div className="tabs">
  <Switch>
    <Route path="/profile" exact component={SelectProfile} />
    <Route
      path="/profile/:profileId"
      render={props => {
        return <Profile {...props} loading={this.state.loading} />;
      }}
    />
  </Switch>
</div>;
```

## Create a Profile Matcher

Now we need to setup our profile matcher. We'll first import `matchPath`, this is the mechanism that `react-router` uses to match paths via the `Route` component.

The first argument is the path to attempt to parse, it will be what you would see in the URL like `/profile/1`. Then we need to pass in our configuration. The keys for the configuration are the same that you would supply to the `Route` component.

```js
import { Switch, Route, Link, matchPath } from "react-router-dom";

//...
const getParams = pathname => {
  const matchProfile = matchPath(pathname, {
    path: `/profile/:profileId`,
  });
  return (matchProfile && matchProfile.params) || {};
};
```

So for our case we want to match

```js
{
  path: `/profile/:profileId`,
}
```

If we were trying to match the root path profile it would require us to use `exact: true` like so.

```js
{
  path: "/profile",
  exact: true,
}
```

Finally we check if we have matched any `params` otherwise we return an empty object.

## ProfileId Change Checking

Now comes the process of checking if we have had anything change and "loading" data in response.

First we need to get both the current and previous pathnames so we can send it to the `getParmas` function we just created.

```js
componentDidUpdate(prevProps, prevState) {
  const { pathname } = this.props.location;
  const { pathname: prevPathname } = prevProps.location;
}
```
We can do that in the `componentDidUpdate` lifecycle method. We have access to the current prop location, as well as the previous location that get passed in via `prevProps`.

Next we will get the params for both paths.

```js
  const { pathname } = this.props.location;
  const { pathname: prevPathname } = prevProps.location;

  const currentParams = getParams(pathname);
  const prevParams = getParams(prevPathname);
```

Then we need to check if they changed. Keep in mind that a potential `pathname` that we will receive is `/profile` without an id, so the params that get returned will be empty. We won't have any data to load.

So to account for all of that we just check that our `currentParams` `profileId` isn't empty, and then we check if they have changed from the `prevParams` to the `currentParams`.
```js
if (currentParams.profileId && currentParams.profileId !== prevParams.profileId) {

}
```

If they have then we trigger our load, and in our case we simply mock a load with a `setState` and a `setTimeout` to clear it after 500 milliseconds.

```js
componentDidUpdate(prevProps, prevState) {
  const { pathname } = this.props.location;
  const { pathname: prevPathname } = prevProps.location;

  const currentParams = getParams(pathname);
  const prevParams = getParams(prevPathname);

  if (currentParams.profileId && currentParams.profileId !== prevParams.profileId) {
    clearTimeout(this.timeout);
    this.setState({
      loading: true,
    });
    setTimeout(() => {
      this.setState({
        loading: false,
      });
    }, 500);
  }
}

componentWillUnmount() {
  clearTimeout(this.timeout);
}
```

## Ending

There you have it. We have a parent route that is matching paths for child rendered routes and doing some data loading. There are other ways to solve this but hopefully this gives you a little bit of an understanding of what is happening inside of React Router. This matching at render time mechanism is also what gives React Router the ability to render dynamic routes at anytime. Change the `path` of a `Route` with any variables and as long as the URL matches it, then it will render.

![](https://images.codedaily.io/lessons/reactRouter/matchPath.gif)
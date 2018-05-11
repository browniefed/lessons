## Intro

With the increasing proliferation of single page apps that control routing, as well as utilize non-cookie methods of authentication there is an ever increasing need to control what routes a user can visit. This should never be the be-all-end-all of security but you should never provide a user an action/route that they can't actually access.

If they somehow access this route you should provide a mechanism that upon validating their identity will bring them right back to what they were attempting to achieve.

![](https://images.codedaily.io/lessons/reactRouter/protectedRoute.gif)

## Setup

We'll setup a simple application to start. It will contain a piece of state that controls whether or not the user has logged in or not.

```js
class App extends Component {
  state = {
    loggedIn: false,
  };
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
        <div className="tabs" />
      </div>
    );
  }
}
```

## Setup Routes

Now we setup our routes below that, we will have our Home and Profile routes. However in our current setup we can navigate to either `Home` or `Profile`. However if there is no user logged in then there is no profile for the user to visit.

Now theoretically this link should be hidden, but if the user navigated to `profile` or authentication expired upon navigating to the `Profile` then we need to tell them that they need to re-authenticate.

```js
<div className="tabs">
  <Switch>
    <Route path="/" exact component={Home} />
    <Route path="/profile" component={Profile} />
  </Switch>
</div>
```

## Create a Protected Component

To accomplish this feature we need to setup a `ProtectedRoute`. It doesn't matter what you call it but the point of this component is to check if the user has permissions to access the page, if not redirect them.

The `ProtectedRoute` is a functional React component, it will take all the same props as a normal `Route` would.

We'll start with a little destructing of the props.

```js
{ component: Comp, loggedIn, path, ...rest }
```

This will rename our `component` to `Comp` so that we can use it to render because React requires components to be capitalized otherwise it will treat it as a normal HTML element.

Next we render a route like usual, but we setup a `render` prop so we can make a determination about what component to render. We render a `Route` at the desired path rather than preventing it from rendering at all because this will allow for a matched route, and thus won't cause issues with non-matching routes in a `Switch`. 

```js
<Route
  path={path}
  {...rest}
  render={props => {

  }}
/>
```
We need to use a `render` prop here because now that we have matched a route we need to do some logic to determine whether or not we should render the `component` that was passed in or redirect the user to another location.

The component looks like this. We just do a ternary with the `loggedIn` prop that will be passed in, and if it's true we can render our `<Comp>`, don't forget to spread the navigation `props` passed into the render prop, otherwise your component won't get the props it expects.

If we aren't logged in then we can redirect back to home, or in most cases you'll redirect to a login page.

```js
const ProtectedRoute = ({ component: Comp, loggedIn, path, ...rest }) => {
  return (
    <Route
      path={path}
      {...rest}
      render={props => {
        return loggedIn ? <Comp {...props} /> : <Redirect to="/" />;
      }}
    />
  );
};
```

## Redirect with State

Now redirecting to the login page is all well and good but what about redirecting them back to where they came from. You may attempt to use the `referrer` or some other mechanism, but what about after a failed login attempt, or if they actually need to signup and go through a complicated flow.

It's imperative to hold on to where they came from and get them right back to what they wanted to do.

```js
const ProtectedRoute = ({ component: Comp, loggedIn, path, ...rest }) => {
  return (
    <Route
      path={path}
      {...rest}
      render={props => {
        return loggedIn ? (
          <Comp {...props} />
        ) : (
          <Redirect
            to={{
              pathname: "/",
              state: {
                prevLocation: path,
                error: "You need to login first!",
              },
            }}
          />
        );
      }}
    />
  );
};
```

We can do that with navigation state. The `to` param of `Redirect` doesn't only work with a string but you can also provide it an object. So in our case we give it a `pathname` of `/` to send them home. As well as a state object with the `prevLocation` as the path they were attempting to visit, and also an `error` message to display. 

This could also be an error code, or boolean like `authFailed: true`, but for simplicity sake well pass back a message.

## Fix Our Route

Now with our `ProtectedRoute` created we need to replace our `Profile` route with our protected component. We'll also need to pass in `loggedIn` from our state otherwise the user will never be able to visit that route.

```js
<ProtectedRoute path="/profile" loggedIn={this.state.loggedIn} component={Profile} />
```

This simple true/false for loggedIn is basic, but you may have a Redux reducer holding onto the users logged in state. The `ProtectedRoute` could then be `connected` and you could inject the users logged in state and then we wouldn't be passing in `loggedIn={this.state.loggedIn}`, it would just be injected by Redux.


## Show Errors

Now we'll show the error message that we passed back. The `state` on `this.props.location` may be undefined in the event that no state was passed to this route. So we'll use the syntax `const { state = {} }` to default during destructure to an empty object.

Then we can render our error message if it exists.

```js
render() {
    const { state = {} } = this.props.location;
    const { error } = state;

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
          {error && <div>ERROR: {error}</div>}
          <Switch>
            <Route path="/" exact component={Home} />
            <ProtectedRoute path="/profile" loggedIn={this.state.loggedIn} component={Profile} />
          </Switch>
        </div>
      </div>
    );
  }
```

## Redirect on Login Success

To setup a mechanism for logging in we add a button called `Login`. It will then call a function to handle logging in. This would likely be an ajax call, but we'll just do a `setState`.

```js
<div className="links">
  <Link to="/" className="link">
    Home
  </Link>
  <Link to="/profile" className="link">
    Profile
  </Link>
  <button onClick={this.handleLogin}>Login</button>
</div>
```

We do our `setState`, and once our `setState` has taken effect which we can guarantee that by using a `componentDidUpdate` callback ref then we can trigger our redirect to our `prevLocation`.

We again grab our `state` from our location and get the `prevLocation`. When we don't have a `prevLocation` we should be sure and setup a default. Our current route is just also `/profile` but you would likely have a landing spot for new users as well as many other protected routes.

```js
handleLogin = () => {
  const { state = {} } = this.props.location;
  const { prevLocation } = state;

  this.setState(
    {
      loggedIn: true,
    },
    () => {
      this.props.history.push(prevLocation || "/profile");
    },
  );
};
```

## Ending

Protecting routes for logged in users is very common, but they can be used for many other experiences. If the user hasn't filled out a profile you could check that and force them to fill in their profile before navigating through your application. 

If a user has completed multiple steps of a wizard but missed a section you could drop them back into the form where they left off. So don't think of this technique as just for login, but think of it as a way to help guide the user through the path to success.

![](https://images.codedaily.io/lessons/reactRouter/protectedRoute.gif)
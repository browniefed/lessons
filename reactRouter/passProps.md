## Intro

There are many reasons that you may want to pass props into a route. If you have data up above and aren't using any external state management libraries you may need to pass down data. Passing data is a common paradigm in React but passing props into a React Router route component requires one extra step.

## What Doesn't Work

Naturally you might want to pass the props with a spread operator onto the `Route` component. This is the normal way that you pass in properties to a React component. However this will not work as the `Route` component will not pass through the props. It will only pass down the props that it provides.

```js
class App extends Component {
  state = {
    message: "You're great!",
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
        <div className="tabs">
          <Switch>
            <Route path="/" exact component={Home} />
            <Route path="/profile" component={Profile} {...this.state} />
          </Switch>
        </div>
      </div>
    );
  }
}
```

So in our case this won't work, if we attempted to access `this.props.message` inside of our `Profile` component we would get `undefined`.

## Pass the Props

In order to pass in properties we need to use the render prop. It's a function that is called and whatever is returned will be rendered for the matched route. The render prop is called with all the properties that would have been spread into the component render from the `component` prop.

```js
<Switch>
  <Route path="/" exact component={Home} />
  <Route path="/profile" render={props => <Profile {...props} {...this.state} />} />
</Switch>
```

So rather than just passing in the `Profile` component we need to return `Profile` as an element with our data from `this.state` spread onto it. Don't forget to also spread our `props` that React Router would have passed in. Otherwise you will not get access to the location, match, history and any other prop you might require.

## Ending

Overall render props are becoming the new normal. They allow for escape hatches to be built into any component library and to allow for the users code to configure and modify what renders without depending on strict rules in the library itself.
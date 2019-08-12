## Introduction

Implementing a sticky header used to be a huge pain before CSS added `position: sticky`. Support for some scenarios (like table headers, etc) are not supported well but you can find out more about those on the caniuse website.

We're going to implement a sticky header that when scrolled to will automatically highlight each section as the user scrolls into them. We'll take advantage of a few different hooks.

## Setup

This will be structure we're going to work with in. A few things to call out are the `top-spacer` and `bottom-spacer` are just arbitrary heights. This allows us to have space to scroll on top and enough room to scroll past the content section completely.

```js
import React, { useRef, useEffect, useState } from "react";
import "./App.css";

function App() {
  return (
    <div className="App">
      <div className="top-spacer" />

      <div className="content">
        <div className="sticky">
          <div className="header">
            <button type="button" className="header_link">
              Leadership
            </button>
            <button type="button" className="header_link">
              Providers
            </button>
            <button type="button" className="header_link">
              Operations
            </button>
          </div>
        </div>
        <div className="section" id="Leadership" />
        <div className="section" id="Providers" />
        <div className="section" id="Operations" />
      </div>

      <div className="bottom-spacer" />
    </div>
  );
}

export default App;
```

Our header is using flexbox and rendering out buttons that will be clickable to scroll to our different sections. Each of our sections will be `40vh` or 40% of the view height, and we'll give each one a different color.

Our `header_link` renders a 3px transparent border, this is because when we apply our `selected` class along with it that has a 3px green bottom border it won't cause the text to jump and be out of line with the other text.

```css
.top-spacer {
  height: 50vh;
}

.bottom-spacer {
  height: 110vh;
}

.header {
  display: flex;
  justify-content: center;
  background-color: #fff;
}

.header_link {
  padding: 20px;
  color: #314d4a;
  font-weight: bold;
  border: none;
  border-bottom: 3px solid transparent;
  cursor: pointer;
  outline: none;
}

.selected {
  border-bottom: 3px solid #11bb9a;
  color: #11bb9a;
}

.section {
  height: 40vh;
}

#Leadership {
  background-color: #a388e8;
}

#Providers {
  background-color: #f4769e;
}

#Operations {
  background-color: #8face0;
}
```

## Make it Sticky

By applying `position: sticky` and giving it a location to sticky to will cause it to stay at that position when the top of the users screen hits that element.

```css
.sticky {
  position: sticky;
  top: 0;
  left: 0;
  right: 0;
  z-index: 10;
}
```

The key bit is that we have placed it inside of another `div`. That div will wrap all of our content. This means the sticky header will be stuck until the user scrolls past the `content`. The content will get its height from the sections that we are rendering.

```js
<div className="content">
  <div className="sticky" />
  {/* articles, other content we care about */}
</div>
```

## Click To Scroll

The browser also implements the ability to scroll to different elements, you can call `element.scrollIntoView()` or also call `scrollTo` on an element to scroll to a specific offset.

We are going to use the `element.scrollIntoView()` since we have references to all of our sections that we want to scroll into view. We define the behavior to smoothly scroll to the start position of the element.

```js
const scrollTo = ele => {
  ele.scrollIntoView({
    behavior: "smooth",
    block: "start",
  });
};
```

We'll get to it next but each section will have it's own `ref` so we can measure the height. But we can also use those refs to the DOM elements to get their offsetTop and scroll to them.

```js
<div className="header">
  <button
    type="button"
    className="header_link"
    onClick={() => {
      scrollTo(leadershipRef.current);
    }}
  >
    Leadership
  </button>
  <button
    type="button"
    className="header_link"
    onClick={() => {
      scrollTo(providerRef.current);
    }}
  >
    Providers
  </button>
  <button
    type="button"
    className="header_link"
    onClick={() => {
      scrollTo(operationsRef.current);
    }}
  >
    Operations
  </button>
</div>
```

## Monitor Scroll

We need to know where the user has currently scrolled to and also be notified when scrolling happens. We can do that by attaching a `scroll` listener to the `window`. We attach it to the window because that is the what is scrolling, if there is an alternate element scrolling you could have attached an `onScroll` prop.

We need to always clean up our effects, and with event listeners when we call `removeEventListener` we need to supply the same function reference to it.

When dealing with hooks, and hooks that will depend on cleanup or hook dependencies it's best practice to create the function inside the `useEffect` hook.

```js
useEffect(() => {
  const handleScroll = () => {};

  window.addEventListener("scroll", handleScroll);
  return () => {
    window.removeEventListener("scroll", handleScroll);
  };
}, []);
```

## Measuring Heights and Offsets

We'll use the `useRef` hook which will allow us to get access to the DOM element of each section so we can get the height/offset. So we have a value to reference for the future we can put them in an array so when we detect something dealing with our `ref` we have a way to identify what thing we're talking about.

```js
const leadershipRef = useRef(null);
const providerRef = useRef(null);
const operationsRef = useRef(null);

const sectionRefs = [
  { section: "Leadership", ref: leadershipRef },
  { section: "Providers", ref: providerRef },
  { section: "Operations", ref: operationsRef },
];
```

Now we attach them to our sections.

```js
  <div className="section" id="Leadership" ref={leadershipRef} />
  <div className="section" id="Providers" ref={providerRef} />
  <div className="section" id="Operations" ref={operationsRef} />
```

In order to know what section we're currently in we need to get the offset top of the element as it sits on the page.

The concept being that if get the `scrollY` (how much the user has scrolled). Then we can see if that value is between the top/bottom of our element.

We use `getBoundingClientRect()` to get the `height` of the element. Then the `offsetTop` will be the top pixel position of it sitting on the page. The bottom will be the `offsetTop + height` of the element.

```js
const getDimensions = ele => {
  const { height } = ele.getBoundingClientRect();
  const offsetTop = ele.offsetTop;
  const offsetBottom = offsetTop + height;

  return {
    height,
    offsetTop,
    offsetBottom,
  };
};
```

## Highlight Section on Scroll

Now that we are monitoring the scroll position, and know the tops and bottoms of each of our sections we can detect if a user is looking at one and highlight it.

We'll need 2 new pieces. The first will be a state variable so we can update and re-render when the user changes sections.

We'll also need the reference to the header so we can measure it's height and adjust our offset since the header will be sitting on top of our content and we want the bottom of the header when it hits a section to cause that section to highlight.

```js
const [visibleSection, setVisibleSection] = useState();
const headerRef = useRef(null);
```

Attach our ref.

```js
<div className="header" ref={headerRef}>
```

First we'll get our header height, we already have a function that will get use the `height` or an element so we can call that and destructure. Our actual scroll position we care about will be the `scrollY` of the window plus the additional height of the header.

```js
const handleScroll = () => {
  const { height: headerHeight } = getDimensions(headerRef.current);
  const scrollPosition = window.scrollY + headerHeight;
};
```

Then we can loop through each of our sections, and get the `offestTop` and `offsetBottom` with out `getDimensions` function. Now we can check if the `scrollPosition` is greater than the top of the element. Aka the user has at least scrolled past it a little. Then we also check if the `scrollPosition` is less than the bottom of the element.

This logic just is checking if we are between the top and bottom. Then we check our `visibleSection` if it is equivalent to the section we found. We want to do this to avoid setting state on every scroll event because we've detected that the user is in the same section.

We use a `find` because we can bail early on finding the item we want, and this also will tell us when we aren't in a section at all.

```js
const selected = sectionRefs.find(({ section, ref }) => {
  const ele = ref.current;
  if (ele) {
    const { offsetBottom, offsetTop } = getDimensions(ele);
    return scrollPosition > offsetTop && scrollPosition < offsetBottom;
  }
});

if (selected && selected.section !== visibleSection) {
  setVisibleSection(selected.section);
}
```

One thing to note is that because we're depending on the previous `visibleSection` we need to add it to our `useEffect` dependencies. When our section changes it will run our clean up, remove the window scroll listener and then re-run our effect.

```js
useEffect(() => {
  const handleScroll = () => {
    const { height: headerHeight } = getDimensions(headerRef.current);
    const scrollPosition = window.scrollY + headerHeight;

    const selected = sectionRefs.find(({ section, ref }) => {
      const ele = ref.current;
      if (ele) {
        const { offsetBottom, offsetTop } = getDimensions(ele);
        return scrollPosition > offsetTop && scrollPosition < offsetBottom;
      }
    });

    if (selected && selected.section !== visibleSection) {
      setVisibleSection(selected.section);
    }
  };

  window.addEventListener("scroll", handleScroll);
  return () => {
    window.removeEventListener("scroll", handleScroll);
  };
}, [visibleSection]);
```

Finally because we have our `visibleSection` state updating when we scroll we can apply our `selected` class to add in a green border bottom and green text to indicate to the user what section they are in.

```js
<div className="header">
  <button
    type="button"
    className={`header_link ${visibleSection === "Leadership" ? "selected" : ""}`}
    onClick={() => {
      scrollTo(leadershipRef.current);
    }}
  >
    Leadership
  </button>
  <button
    type="button"
    className={`header_link ${visibleSection === "Providers" ? "selected" : ""}`}
    onClick={() => {
      scrollTo(providerRef.current);
    }}
  >
    Providers
  </button>
  <button
    type="button"
    className={`header_link ${visibleSection === "Operations" ? "selected" : ""}`}
    onClick={() => {
      scrollTo(operationsRef.current);
    }}
  >
    Operations
  </button>
</div>
```

## Remove First Highlight

One other bit we need to take care of is if the user scrolls into our content, and then scrolls back to the top of the page. If we add an else if and detect when we haven't found a `selected` section. That means we aren't in any section at all. Then we check if we have a section selected, and can remove our selection by setting `visibleSection` to undefined.

```js
if (selected && selected.section !== visibleSection) {
  setVisibleSection(selected.section);
} else if (!selected && visibleSection) {
  setVisibleSection(undefined);
}
```

## Scroll Position Restoration

If the user had scroll to a position on the page and refreshes the page, most browsers will restore the position the user was scrolled to. So in our `useEffect` hook if we call our scroll function it will run all the logic based on the current scroll position and update our selected state accordingly.

```js
useEffect(() => {
  const handleScroll = () => {
    // Scroll Code
  };

  handleScroll();
  window.addEventListener("scroll", handleScroll);
  return () => {
    window.removeEventListener("scroll", handleScroll);
  };
}, [visibleSection]);
```

## End

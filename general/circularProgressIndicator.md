# Intro

# Setup our Holding Circle

```css
.wrapper {
  position: relative;
  width: 600px;
  height: 600px;
  border-radius: 300px;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2);
  background-color: #fff;
}
```

```js
import React, { useEffect, useState } from "react";
import "./App.css";

function App() {
  return (
    <div className="App">
      <div className="wrapper" />
    </div>
  );
}

export default App;
```

# Create a Progress Tick

```js
const [progress, setProgress] = useState(0);

useEffect(() => {
  setTimeout(() => {
    const nextProgress = progress + 0.05;
    if (nextProgress > 1) {
      return;
    }

    setProgress(nextProgress);
  }, 1000);
});
```

# Setup our SVG

```js
<svg viewBox="0 0 50 50" width="600px" height="600px" className="circle-progress">
  <circle className="progress" fill="transparent" />
</svg>
```

```css
.circle-progress {
  position: absolute;
  left: 0;
  top: 0;
}
```

# Understanding Stroke Dash Array and Offset

# Calculate Position

```js
const position = Math.min(progress, 1);
const complete = position === 1;
const notMoved = position === 0;
```

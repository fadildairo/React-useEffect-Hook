# React-useEffect-Hook
React's useEffect hook allows us to perform [side effects](https://www.reddit.com/r/reactjs/comments/8avfej/what_does_side_effects_mean_in_react/) in our components. The useEffect hook is often used for interactions with the Browser/DOM API or external API like data fetching. I'll demonstrate how the useEffect hook is used for interaction with the Browser API by implementing a simple stopwatch.
```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

  return (
    <div>
      {!isOn && (
        <button type="button" onClick={() => setIsOn(true)}>
          Light On
        </button>
      )}

      {isOn && (
        <button type="button" onClick={() => setIsOn(false)}>
          Light Off
        </button>
      )}
    </div>
  );
}

export default App;
``` 
The stopwatch has not been implemented yet. This is just a [conditional rendering](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND) to show either a "Light On" or "Light Off" button. Moving on, let me introduce a setInterval() method that calls a function at specified intervals (in milliseconds). The function used for the interval logs 'tick' every second to the developer tools of your browser:
```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

/*Here*/
React.useEffect(() => {
    setInterval(() => console.log('tick'), 1000);
  });

  return (
    <div>
      {!isOn && (
        <button type="button" onClick={() => setIsOn(true)}>
          Light On
        </button>
      )}

      {isOn && (
        <button type="button" onClick={() => setIsOn(false)}>
          Light Off
        </button>
      )}
    </div>
  );
}

export default App;
``` 
It's worth noting that the setInterval() method continues calling the function until clearInterval() is called, or the window is closed. In order to remove the interval when the component unmounts i.e when the component gets destroyed or unmounted from the DOM (but also after every other [render](https://www.reddit.com/r/reactjs/comments/vsl6ng/what_is_meant_by_render_in_react_context/) update), you can return a function in useEffect for anything to be called for the cleanup. This is useful to avoid [memory leaks](https://en.wikipedia.org/wiki/Memory_leak) when the component isn't there anymore.

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

  React.useEffect(() => {
    const interval = setInterval(() => console.log('tick'), 1000);
    /*Here*/
    return () => clearInterval(interval);
  });

  ...
}

export default App;
``` 
useEffect runs on every render. When we click on  'Light On' / 'Light Off' button, the state changes which then triggers another effect. If you would log how many times the function within the effect is called, you would see that it sets a new interval every time the state of the component changes:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

  React.useEffect(() => {
    /*Here*/
    console.log('effect runs');
    const interval = setInterval(() => console.log('tick'), 1000);
   
    return () => clearInterval(interval);
  });

  ...
}

export default App;
``` 
In order to only run the effect on mount and unmount of the component, you can add an empty array. You'll notice the effect only runs on the first render by checking the browser console:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

  React.useEffect(() => {
    const interval = setInterval(() => console.log('tick'), 1000);

    return () => clearInterval(interval);
  }, []);  /*Here*/

  ...
}

export default App;
``` 

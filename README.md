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
The effect is cleaned up when the browser refreshes. But we can also set the effect to run only when the isOn variable changes. Only when one of the variables in the array changes, the effect will run during the update cycle i.e componentDidUpdate(). If you keep the array empty, the effect will only run on mount and unmount, because there is no variable to be checked for running the side-effect again:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

  React.useEffect(() => {
    const interval = setInterval(() => console.log('tick'), 1000);

    return () => clearInterval(interval);
  }, [isOn]); /*Here*/

  ...
}

export default App;
``` 
With isOn added to the array, the interval keeps running whether isOn boolean is true or false. We want add a condition to only run the interval when the stopwatch is activated:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);

  React.useEffect(() => {
    let interval; /*Here*/

    if (isOn) { /*Here*/
      interval = setInterval(() => console.log('tick'), 1000);
    }

    return () => clearInterval(interval);
  }, [isOn]);

  ...
}

export default App;
``` 
We can now introduce another state to keep track of the timer of the stopwatch. It is used to update the timer, but only when the stopwatch is activated:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);
  const [timer, setTimer] = React.useState(0); /*Here*/

  React.useEffect(() => {
    let interval;

    /*Here*/
      interval = setInterval(
        () => setTimer(timer+ 1), 
        1000,
      );
    }

    return () => clearInterval(interval);
  }, [isOn]);

  return (
    <div>
      {timer} {/*Here*/}

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
There is still a problem in the code. In order to always receive the latest state for the timer when the interval is running and not the stale state, we can use a functional update 'setTimer(t => t + 1)' since we only need 'timer' in the 'setTimer' call:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);
  const [timer, setTimer] = React.useState(0);

  React.useEffect(() => {
    let interval;

    if (isOn) {
      interval = setInterval(
        () => setTimer(timer => timer + 1), /*Here*/ 
        1000,
      );
    }

    return () => clearInterval(interval);
  }, [isOn]);

  ...
}

export default App;
``` 
An alternative would have been to run the effect also when the timer changes. Then the effect would receive the latest timer state:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);
  const [timer, setTimer] = React.useState(0);

  React.useEffect(() => {
    let interval;

    if (isOn) {
      interval = setInterval(
        () => setTimer(timer + 1),
        1000,
      );
    }

    return () => clearInterval(interval);
  }, [isOn, timer]); /*Here*/ 

  ...
}

export default App;
``` 
Tha Daa! Okay, that's the implementation for the stopwatch that uses the Browser API. We can also extend it by adding a "Reset" button:

```javascript
import React from 'react';

function App() {
  const [isOn, setIsOn] = React.useState(false);
  const [timer, setTimer] = React.useState(0);

  React.useEffect(() => {
    let interval;

    if (isOn) {
      interval = setInterval(
        () => setTimer(timer => timer + 1),
        1000,
      );
    }

    return () => clearInterval(interval);
  }, [isOn]);

  /*Here*/
  const onReset = () => {
    setIsOn(false);
    setTimer(0);
  };

  return (
    <div>
      {timer}

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

      {/*Here*/}
      <button type="button" disabled={timer === 0} onClick={onReset}>
        Reset
      </button>
    </div>
  );
}

export default App;
``` 
That's it. The useEffect hook is used for side-effects in React function components that are used for interacting with the Browser/DOM API or other third-party APIs (e.g. data fetching). You can read more about [the useEffect hook in React's documentation.](https://reactjs.org/docs/hooks-effect.html)

Reference: https://www.robinwieruch.de/react-hooks/

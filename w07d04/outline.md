# External Resources

### Packages Needed

* `yarn add @testing-library/react-hooks react-test-renderer axios`

# Outline

### Custom Hooks
- From the [React Docs](https://reactjs.org/docs/hooks-custom.html):
> Building your own Hooks lets you extract component logic into reusable functions.
- We can pull repetitive or complex code out of our components and move it into _custom hooks_
- _Custom hooks_ are just JavaScript functions that can use React hooks
- They must start with the prefix `use` so that React knows they are hooks
- Multiple components using the same custom hook **do not share state**

### Create a component called `Title` with logic to change the document title

```jsx
const Title = () => {
  const [title, setTitle] = React.useState('Hooks are neato!');

  React.useEffect(() => {
    document.title = title;
  }, [title]);

  React.useEffect(() => {
    setTimeout(() => {
      setTitle('I luv hooks!');
    }, 3000);
  }, []);

  return (
    <h1>Document Title</h1>
  );
};
```

### Create a `useDocumentTitle` hook

```jsx
const useDocumentTitle = (title) => {
  React.useEffect(() => {
    document.title = title;
  }, [title]);
};
```

### Refactor the `Title` component to use the new hook

```jsx
useDocumentTitle(title);
```

### Create a component called `Mouse` with logic to listen for the mouse position

```jsx
const Mouse = () => {
  const [coords, setCoords] = React.useState({ x: 0, y: 0 });

  React.useEffect(() => {
    document.addEventListener('mousemove', e => {
      setCoords({ x: e.clientX, y: e.clientY});
    });
  }, []);

  const style = {
    fontSize: `${coords.y / 5}px`
  };

  return (
    <div>
      <h2 style={style}>Mouse Position</h2>
      <h2>X: {coords.x} Y: {coords.y}</h2>
    </div>
  );
};
```

### Show `useMousePosition.test.js` and unskip the tests

### Create `useMousePosition.js` and move the mouse logic into it

```js
import React from 'react';

const useMousePosition = () => {
  const [coords, setCoords] = React.useState({ x: 0, y: 0 });

  React.useEffect(() => {
    const mouseHandler = (event) => {
      setCoords({ x: event.clientX, y: event.clientY });
    };

    document.addEventListener('mousemove', mouseHandler);

    return () => document.removeEventListener('mousemove', mouseHandler);
  }, []);

  return coords;
};

export default useMousePosition;
```

### Import the new hook into the `Mouse` component

```js
import useMousePosition from '../hooks/useMousePosition';

const coords = useMousePosition();

const style = {
  fontSize: `${coords.y / 5}px`,
  color: `rgb(${coords.x / 4}, ${coords.y / 4}, 0)`,
  backgroundColor: `rgb(0, ${coords.x / 4}, ${coords.y / 4})`,
  padding: '10px',
  border: `10px dotted rgb(${coords.y / 4}, 0, ${coords.x / 4})` 
};
```

### Import the hook into the `Title` component

```js
const Title = () => {
  const coords = useMousePosition();
  const title = `Mouse is at ${coords.x}, ${coords.y}`;
  useDocumentTitle(title);

  return (
    <div>
      <h1>Document Title</h1>
    </div>
  );
};
```

### Create the `Input` component and give it some form handling logic

```jsx
import React from 'react';

const Input = () => {
  const [username, setUsername] = React.useState('');
  const [password, setPassword] = React.useState('');

  const handleSubmit = (event) => {
    event.preventDefault();
    alert(`Thanks for logging in ${username} with password: ${password}`);
  };

  const onChangeUsername = (event) => {
    setUsername(event.target.value);
  };

  const onChangePassword = (event) => {
    setPassword(event.target.value);
  };

  return (
    <div>
      <h2>Input Hook</h2>
      <form onSubmit={handleSubmit}>
        <label htmlFor="username">Username:</label>
        <input 
          type="text"
          id="username"
          value={username}
          onChange={onChangeUsername}
        />
        <br/>
        <label htmlFor="password">Password:</label>
        <input 
          type="text"
          id="password"
          value={password}
          onChange={onChangePassword}
        />
        <br/>
        <button type="submit">Login!</button>
      </form>
    </div>
  );
};

export default Input;
```

### Show `useInput.test.js`

### Create `useInput.js` hook

```js
import React from 'react';

const useInput = (initialValue) => {
  const [value, setValue] = React.useState(initialValue);

  const onChange = (event) => {
    setValue(event.target.value);
  };

  return { value, onChange };
};

export default useInput;
```

### Refactor the `Input` component to use the new hook

```jsx
import React from 'react';
import useInput from '../hooks/useInput';

const Input = () => {
  const usernameInput = useInput('');
  const passwordInput = useInput('');

  const handleSubmit = (event) => {
    event.preventDefault();
    alert(`Thanks for logging in ${usernameInput.value} with password: ${passwordInput.value}`);
  };

  return (
    <div>
      <h2>Input Hook</h2>
      <form onSubmit={handleSubmit}>
        <label htmlFor="usernameInput">Username:</label>
        <input 
          type="text"
          id="username"
          {...usernameInput}
        />
        <br/>
        <label htmlFor="password">Password:</label>
        <input 
          type="password"
          id="password"
          {...passwordInput}
        />
        <br/>
        <button type="submit">Login!</button>
      </form>
    </div>
  );
};

export default Input;
```

### Discuss how the value could be reset inside the hook to extend the functionality

### Create the `useLocationData` hook

```js
const useLocationData = () => {
  const [coords, setCoords] = React.useState({ lat: 0, lon: 0 });

  React.useEffect(() => {
    window.navigator.geolocation.getCurrentPosition((navInfo) => {
      setCoords({ lat: navInfo.coords.latitude, lon: navInfo.coords.longitude });    
    });
  }, []);

  return coords;
};
```

### Create the `Location` component to use the new hook

```jsx
import useLocationData from '../hooks/useLocationData';

const Location = () => {
  const coords = useLocationData();

  const buttonClickHandler = () => {
    const message = `lat: ${coords.lat.toFixed(2)}, lon: ${coords.lon.toFixed(2)}`;
    alert(message);
  };

  return (
    <button onClick={buttonClickHandler}>See Your Location!</button>
  );
};
```

### Review `useRequest` hook

```jsx
import React from 'react';
import useRequest from '../hooks/useRequest';

const Request = () => {
  const { data, loading } = useRequest('https://www.dnd5eapi.co/api/classes');

  return (
    <div>
      <h2>D&D Classes</h2>
      { loading && <p>Please wait...</p> }
      { data.results && data.results.map(datum => (
        <p key={datum.index}>{ datum.name }</p>
      )) }
    </div>
  );
};

export default Request;
```

### Review `useKeyPress` hook

```jsx
import React from 'react';
import useKeyPress from '../hooks/useKeyPress';

const KeyPress = () => {
  const happyPress = useKeyPress('h');
  const sadPress = useKeyPress('s');

  return (
    <div>
      { happyPress && <h2>Smile</h2> }
      { sadPress && <h2>Frown</h2> }
    </div>
  );
};

export default KeyPress;
```

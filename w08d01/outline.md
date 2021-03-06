### Overview
- This lecture has 90 mins of live-coding
- Ensure the theory portion goes no longer than 30 mins

### Important Links
- [Jest Homepage](https://jestjs.io/)
- [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/intro)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)
- [JestDOM](https://github.com/testing-library/jest-dom)
- [Which query should I use?](https://testing-library.com/docs/guide-which-query)

### Tools for testing React
- [Jest](https://jestjs.io/)
  * Jest is the framework we use to run our tests
  * Comes with `create-react-app`, so no need to configure
  * `npm run test` will start Jest in watch mode and run the tests
- [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/intro)
  * A set of tools to help target DOM elements and trigger DOM events
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)
  * Built on top of the DOM Testing Library, gives us more possibilities to target and render React elements to make them possible to test
- [JestDOM](https://github.com/testing-library/jest-dom)
  * JestDOM is a set of matchers (like `.toHaveClass()` or `.toBeVisible()`) to help target elements in the DOM

### Add Features to App Following TDD
- helper functions (unit tests)
  - choose a valid response for the computer player (currently hard-coded)
  - determine the status message to display when the game is done
- features (integration tests)
  - clicking on the robot head will toggle the cheating boolean
  - render appropriate message in Result component based on game status

```bash
% npm test -- --verbose
% yarn test --verbose

% npm test -- --coverage
% yarn test --coverage

% npm test -- --coverage --watchAll=false
% yarn test --coverage --watchAll=false
```

1. `src/helpers/__tests__/helpers.test.js`

```js
describe('chooseRobotItem function', () => {
  test('given player choice and cheating is true, returns winning choice', () => {
    const cheating = true;

    let playerSelection = 'Axe';
    let result = chooseRobotItem(cheating, playerSelection);
    expect(result).toBe('Moai');

    playerSelection = 'Moai';
    result = chooseRobotItem(cheating, playerSelection);
    expect(result).toBe('Tree');

    playerSelection = 'Tree';
    result = chooseRobotItem(cheating, playerSelection);
    expect(result).toBe('Axe');
  });

  test('given player choice and cheating is false, returns a valid choice', () => {
    const cheating = false;
    const playerSelection = 'Axe';
    
    const result = chooseRobotItem(cheating, playerSelection);
    const options = ['Axe', 'Tree', 'Moai'];
    // expect(options.includes(result)).toBeTruthy();
    expect(options).toContain(result);
  });
});
```

2. `src/helpers/helpers.js`

```js
export const chooseRobotItem = (cheating, playerItem) => {
  const lookup = {
    'Tree': 'Axe',
    'Moai': 'Tree',
    'Axe': 'Moai'
  };
  if (cheating) {
    return lookup[playerItem];
  } else {
    const choices = ["Moai", "Axe", "Tree"];
    const randomIndex = Math.floor(Math.random() * choices.length);
    return choices[randomIndex];
  }
};
```

3. `src/helpers/__tests__/helpers.test.js`

```js
describe('genFeedbackMessage function', () => {
  test('returns correct message when given a status', () => {
    const loss = genFeedbackMessage('Lost');
    const win = genFeedbackMessage('Won');
    const tie = genFeedbackMessage('Tied');
    const waiting = genFeedbackMessage('Waiting');

    expect(loss).toEqual('You lost!');
    expect(win).toEqual('Good job!');
    expect(tie).toEqual('Tie game!');
    expect(waiting).toEqual('Waiting for your choice!');
  });
});
```

4. `src/helpers/helpers.js`

```js
export const genFeedbackMessage = (status) => {
  switch (status) {
    case 'Won':
      return 'Good job!';
    case 'Lost':
      return 'You lost!';
    case 'Tied':
      return 'Tie game!';
    default:
      return 'Waiting for your choice!';
  }
};
```

5. `src/components/__tests__/Game.test.jsx`

```js
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import Game from '../Game';

test('change cheat state when clicking on robot', () => {
  const { getByTestId } = render(<Game />);
  const robotIcon = getByTestId('robot-icon');

  fireEvent.click(robotIcon);
  expect(robotIcon).toHaveClass('cheating');

  fireEvent.click(robotIcon);
  expect(robotIcon).not.toHaveClass('cheating');
});
```

6. `src/components/Computer.jsx`

```js
const handleClick = () => {
  return setState(prevState => (
    { ...prevState, cheating: !prevState.cheating }
  ));
};

<span
  data-testid="robot-icon"
  role="img" 
  aria-label="robot" 
  className={state.cheating ? "cheating" : null}
  onClick={handleClick}
>
  🤖
</span>
```

7. `src/components/__tests__/Result.test.js`

```js
// convert this test to use beforeEach and write out the tests for other statuses
test('shows appropriate message when the status is "Waiting"', () => {
  const fakeState = {
    compSelection: null,
    playerSelection: null,
    status: 'Waiting',
    cheating: false
  };
  
  const { container } = render(<Result status={fakeState.status} />);

  expect(getByTestId(container, 'result_footer')).toHaveTextContent('Waiting for your choice!');
});
```

```js
let fakeState;

beforeEach(() => {
  fakeState = {
    compSelection: null,
    playerSelection: null,
    status: 'Waiting',
    cheating: false
  };
});

test('shows appropriate message when the status is "Waiting"', () => {
  fakeState.status = 'Waiting';
  const { container } = render(<Result status={fakeState.status} />);

  expect(getByTestId(container, 'result_footer')).toHaveTextContent('Waiting for your choice!');
});
```

8. Demonstrate `prettyDOM` and `debug`

```js
import { render, getByTestId, prettyDOM } from '@testing-library/react';
console.log(prettyDOM(container));

const { container, debug } = render(<Result status={fakeState.status} />);
debug();
```

9. Mock `axios`

```js
// src/components/__tests__/Result.test.js
import React from 'react';
import { render, getByTestId, fireEvent } from '@testing-library/react';
import Result from '../Result';
import axios from 'axios';

jest.mock('axios');

const data = {
  resultCount: 3,
  results: [
    { id: 1, name: 'Alice', score: 10 },
    { id: 2, name: 'Bob', score: 5 },
    { id: 3, name: 'Carol', score: 2 }
  ]
};

// this function must be marked as `async` in order to use `await` within it
test('Axios test', async () => {
  const { getByTestId, findByText } = render(<Result status="Waiting" />);
  const button = getByTestId('fetch-highscores');

  // mock any calls to axios.get with hardcoded return value `data`
  axios.get.mockResolvedValueOnce({ data });

  // firing this event causes the axios call to happen
  fireEvent.click(button);

  // findBy functions return a promise which we can `await`
  await findByText('Bob');

  // we could get jest to wait instead
  return expect(findByText('Bob')).resolves.toBeTruthy();

  // or we could return a promise
  return findByText('Bob');
});
```

10. Update the Result component to make the AJAX request on button click

```jsx
const [highScores, setHighScores] = React.useState([]);

const fetchHighScores = () => {
  axios
    .get('/high-scores')
    .then(response => setHighScores(response.data.results))
    .catch(err => console.error(err));
};

return(
  <footer data-testid="result_footer">
    <h2>Waiting for your choice!</h2>
    <button onClick={fetchHighScores} data-testid="high-scores">High Scores!</button>
    { highScores.map(highScore => <li key={highScore.id}>{highScore.name}</li>) }
  </footer>
);
```

11. Function mock example

```js
const mock = jest.fn();
let result = mock('foo');

expect(result).toBeUndefined();
expect(mock).toHaveBeenCalled();
expect(mock).toHaveBeenCalledTimes(1);
expect(mock).toHaveBeenCalledWith('foo');

const mockTwo = jest.fn(() => 'bar');
result = mockTwo('foo');

expect(result).toBe('bar');
```

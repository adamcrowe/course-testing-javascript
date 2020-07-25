# course-testing-javascript
* Course: [Frontend Masters: JavaScript Testing Practices and Principles](https://frontendmasters.com/courses/testing-practices-principle) by Kent C. Dodds
* Course: [Frontend Masters: Testing React Applications v2](https://frontendmasters.com/courses/testing-react) by Kent C. Dodds
  * [Repository](https://github.com/kentcdodds/testing-workshop)

## TEST DRIVEN DEVELOPMENT (TDD)
* RED -> GREEN -> REFACTOR -> RED...
    * RED: Write test that fails then...
    * GREEN: Write code to pass test then...
    * REFACTOR: Iterate test and code to implement feature

## TOOLS
* Unit Testing with [Jest](https://jestjs.io)
* Integration Testing with [Jest](https://jestjs.io)
* Integration Testing for React with [React Testing Library](https://testing-library.com/react)
    * Replacement for [Enzyme](https://enzymejs.github.io/enzyme)
* End to End (E2E) Testing with [Cypress](https://www.cypress.io)
    * Replacement for [Selenium](https://www.selenium.dev)

## TESTING TIPS
> The more your tests resemble the way your software is used, the more confidence they can give you. -- [React Testing Library](https://testing-library.com)
* Write test cases that simulate, as closely as possible, how an actual user (or manual tester) would use the system
* Write and run tests before writing the code so that you can verify the test is running (and failing first time)
* [Co-locate](https://kentcdodds.com/blog/colocation) tests closely with code so they are easy to find and update
    * Unit tests live in the same directory as the code
    * Integration tests live in a common/parent directory
    * End-to-End tests live in the root directory
* Write test case descriptions to include test subject to make them easy to identity in test logs:

```javascript
describe('isPasswordAllowed only allows some passwords', () => {
    const allowedPasswords = []
    const disallowedPasswords = []

    allowedPasswords.forEach(password => {
        test(`allows ${password}`, () => { // test case description includes test subject (password)
            expect(isPasswordAllowed(password)).toBe(true)
        })
    })

    disallowedPasswords.forEach(password => {
        test(`disallows ${password}`, () => {
            expect(isPasswordAllowed(password)).toBe(true)
        })
    })
})
```

## JEST MOCKS/SPIES
* The optimal strategy is to mock an entire module (`__mocks__` directory) rather than monkey-patching (`jest.mock()`)

### Mock Directory Example
* Mock `axios.js` in `__mocks__` directory:

```javascript
const defaultResponse = { data: { response: 'hello world' } };
module.exports = { post: jest.fn(() => Promise.resolve(defaultResponse)) };
```

### Monkey Patching Example
* [Mock Functions](https://jestjs.io/docs/en/mock-function-api)
* [jest.spyOn()](https://jestjs.io/docs/en/jest-object#jestspyonobject-methodname)
* [jest.mock()](https://jestjs.io/docs/en/jest-object#jestmockmodulename-factory-options)

```javascript
// Using jest.spyOn
jest.spyOn(utils, 'myFunc'); // spy on module myFunc
utils.myFunc.mockImplementation((p1, p2) => p2); // replace myFunc
expect(utils.myFunc.mock.calls).toEqual(//...);
utils.myFunc.mockRestore(); // restore original myFunc
```

```javascript
// Using jest.mock
jest.mock(
    '../utils', // mock module
    () => {
        return {
            myFunc: jest.fn((p1, p2) => p2)); // replace myFunc
        }
    }
);
```

## REACT TESTING
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import ItemList from '../item-list';

test('renders the items in a list', () => {
    const container = document.createElement('div');
    ReactDOM.render(
        <ItemList items={['apple', 'orange', 'pear']} />,
        container,
    );
    expect(container.textContent).toMatch('apple');
    expect(container.textContent).toMatch('orange');
    expect(container.textContent).toMatch('pear');
});
```

```javascript
// using React DOM
import React from 'react';
import ReactDOM from 'react-dom';
import Login from '../login';

test('calls onSubmit with the username and password when submitted', () => {
    const fakeUser = { username: 'chucknorris', password: '(╯°□°）╯︵ ┻━┻' }
    const handleSubmit = jest.fn(); // mock
    const div = document.createElement('div');
    ReactDOM.render(<Login onSubmit={handleSubmit} />, div);

    const inputs = div.querySelectorAll('input');
    const usernameNode = inputs[0];
    const passwordNode = inputs[1];
    const formNode = div.querySelector('form');
    const submitButtonNode = div.querySelector('button');

    usernameNode.value = fakeUser.username;
    passwordNode.value = fakeUser.password;

    const event = new window.Event('submit');
    formNode.dispatchEvent(event);

    expect(handleSubmit).toHaveBeenCalledTimes(1);
    expect(handleSubmit).toHaveBeenCalledWith(fakeUser);
    expect(submitButtonNode.type).toBe('submit');
});
```

```javascript
// Using RTL
import React from 'react'
import { generate } from 'til-client-test-utils'
import { render, Simulate } from 'react-testing-library'
import Login from '../login'

test('calls onSubmit with the username and password when submitted', () => {
    const fakeUser = generate.loginForm()[];
    const handleSubmit = jest.fn();
    const {container, getByLabelText, getByText} = render(
        <Login onSubmit={handleSubmit} />,
    );

    const usernameNode = getByLabelText('username');
    const passwordNode = getByLabelText('password');
    const formNode = container.querySelector('form');
    const submitButtonNode = getByText('submit');

    usernameNode.value = fakeUser.username;
    passwordNode.value = fakeUser.password;
    Simulate.submit(formNode);

    expect(handleSubmit).toHaveBeenCalledTimes(1);
    expect(handleSubmit).toHaveBeenCalledWith(fakeUser);
    expect(submitButtonNode.type).toBe('submit');
})
```

```javascript
// Integration test
import React from 'react'
import { Simulate } from 'react-dom/test-utils'
import axiosMock from 'axios'
import { renderWithRouter, generate } from 'til-client-test-utils'
import { init as initAPI } from '../utils/api'
import App from '../app'

beforeEach(() => {
    window.localStorage.removeItem('token')
    axiosMock.__mock.reset()
    initAPI()
})

test('register a new user', async () => {
    const {
        container,
        getByTestId,
        getByText,
        finishLoading,
        getByLabelText,
    } = renderWithRouter(<App />)

    // wait for the app to finish loading the mocked requests
    await finishLoading()

    // navigate to register
    const leftClick = {button: 0}
    Simulate.click(getByText('Register'), leftClick)
    expect(window.location.href).toContain('register')

    // fill out form
    const fakeUser = generate.loginForm()
    const usernameNode = getByLabelText('Username')
    const passwordNode = getByLabelText('Password')
    const formWrapper = container.querySelector('form')
    usernameNode.value = fakeUser.username
    passwordNode.value = fakeUser.password

    // submit form
    const {post} = axiosMock.__mock.instance
    const token = generate.token(fakeUser)
    post.mockImplementationOnce(() =>
        Promise.resolve({
        data: {user: {...fakeUser, token}},
        }),
    )
    Simulate.submit(formWrapper)

    // wait for the mocked requests to finish
    await finishLoading()

    // assert calls
    expect(axiosMock.__mock.instance.post).toHaveBeenCalledTimes(1)
    expect(axiosMock.__mock.instance.post).toHaveBeenCalledWith(
        '/auth/register',
        fakeUser,
    )

    // assert state of app
    expect(window.localStorage.getItem('token')).toBe(token)
    expect(window.location.href).not.toContain('register')
    expect(getByTestId('username-display').textContent).toEqual(fakeUser.username)
    expect(getByText('Logout')).toBeTruthy()
})
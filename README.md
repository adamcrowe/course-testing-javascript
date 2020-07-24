# course-testing-javascript
* Course: [Frontend Masters: JavaScript Testing Practices and Principles](https://frontendmasters.com/courses/testing-practices-principle) by Kent C. Dodds
  * [Repository](https://github.com/kentcdodds/testing-workshop)

## TEST DRIVEN DEVELOPMENT (TDD)
* RED -> GREEN -> REFACTOR -> RED...
    * RED: Write test that fails then...
    * GREEN: Write code to pass test then...
    * REFACTOR: Iterate test and code to implement feature

## TESTING TIPS
* Write and run tests before writing the (failing) code so that you can verify the test is running
* [Co-locate](https://kentcdodds.com/blog/colocation) tests closely with code so they are easy to find and update
    * Unit tests live in the same directory as the code
    * Integration tests live in a common/parent directory
    * End-to-End tests live in the root directory
* Write test descriptions to include test subject to make them easy to identity in test logs:

```javascript
describe('isPasswordAllowed only allows some passwords', () => {
    const allowedPasswords = []
    const disallowedPasswords = []

    allowedPasswords.forEach(password => {
        test(`allows ${password}`, () => { // description includes test subject (password)
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
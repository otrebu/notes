---
title: testing
keywords: testing
tags: testing, jest 
---

## Running tests
`package.json`
```
"test:watch": "jest --watch"
```

Interactive watch, you can just run failing
tests which will reduce the console output.

### Running test useful packages:

`jest-watch-select-projects` package to select projects on watch. 
`jest-watch-typeahead`: Filter your tests by file name or test name.
Config:
``` watchPlugins: [
    'jest-watch-typeahead/filename',
    'jest-watch-typeahead/testname',
    'jest-watch-select-projects',
  ],
```
### Running test pre-commit:

Lint-stage and husky config:

`package.json`
  ```
  "lint-staged": {
    "**/*.+(js|json|css|html|md)": [
      "prettier",
      "jest --findRelatedTests",
      "git add"
    ]
  },
  ```

## Debug tests
`package.json`
```
"test:debug": "node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand --watch"
```

## Test coverage

`package.json`
```
"test:coverage": "jest --coverage"
```
`jest.config.js`
```
collectCoverageFrom: ['**/src/**/*.js'],
coverageThreshold: {
    global: {
      statements: 15,
      branches: 10,
      functions: 15,
      lines: 15,
    },
    './src/shared/utils.js': {
      statements: 100,
      branches: 80,
      functions: 100,
      lines: 100,
    },
  },
```

Add coverage folder to `.gitignore` to not include in repo.

## Configure Jest

You can add projects with different configurations:
`jest.config.js`
``` 
projects: [
    './test/jest.lint.js',
    './test/jest.client.js',
    './test/jest.server.js',
    './server',
  ],
```

Add a label:

`./test/jest.server.js`
```
  displayName: "server"
```



## Using jest

Clear mocks between tests:
```
afterEach(() => {
  jest.clearAllMocks()
})
```

Mock a function and restore between tests:
```
// mock a function before test run
beforeAll(() => {
  jest.spyOn(console, 'error').mockImplementation(() => {})
})
// restore the function after all test run
afterAll(() => {
  console.error.mockRestore()
})
// clears mock calls data between tests
afterEach(() => {
  jest.clearAllMocks()
})
```

## Testing react with `@testing-library/react`

`@testing-library/jest-dom`

A set of custom jest matchers that you can use to extend jest. These will make your tests more declarative, clear to read and to maintain.

[Queries](https://testing-library.com/docs/queries/about) are methods that Testing Library gives you to find elements on the page.

Simple example:

```
import React from 'react'
import {render} from '@testing-library/react'
import {FavoriteNumber} from '../favorite-number'

test('renders a number input with a label "Favorite Number"', () => {

    // returns the queries for that component
  const {getByLabelText} = render(<FavoriteNumber />)
  const input = getByLabelText(/favorite number/i)
  expect(input).toHaveAttribute('type', 'number')
})
```

You can import `debug` function to use from the render.

### Wrapping components 
` ` option available in render method arguments.

Example:
```
 const {rerender, getByText, queryByText, getByRole, queryByRole} = render(
    <Bomb />,
    {wrapper: ErrorBoundary},
  )
```


### Testing events

Firing an event like the user would using this  companion library ( [https://testing-library.com/docs/ecosystem-user-event/
](@testing-library/user-event) ) for Testing Library that provides more advanced simulation of browser interactions than the built-in fireEvent method.

Example:

```
import React from 'react'
import user from '@testing-library/user-event'
import {render} from '@testing-library/react'
import {FavoriteNumber} from '../favorite-number'

test('entering an invalid value shows an error message', () => {
  const {getByLabelText, getByRole} = render(<FavoriteNumber />)
  const input = getByLabelText(/favorite number/i)
  user.type(input, '10')
  expect(getByRole('alert')).toHaveTextContent(/the number is invalid/i)
})
```

### Props updates

Example:

```
import React from 'react'
import user from '@testing-library/user-event'
import {render} from '@testing-library/react'
import {FavoriteNumber} from '../favorite-number'

test('entering an invalid value shows an error message', () => {
  const {getByLabelText, getByRole, queryByRole, rerender} = render(
    <FavoriteNumber />,
  )
  const input = getByLabelText(/favorite number/i)
  user.type(input, '10')
  expect(getByRole('alert')).toHaveTextContent(/the number is invalid/i)
  rerender(<FavoriteNumber max={10} />)
  // query returns null rather than an error if the
  // element is not found
  expect(queryByRole('alert')).toBeNull()
})
```

### Testing accessibility

Use `jest-axe` 
Configure [it](https://github.com/nickcolley/jest-axe#testing-react-with-react-testing-library).

```
import {axe} from 'jest-axe'
...
test('accessible forms pass axe', async () => {
  const {container} = render(<AccessibleForm />)
  expect(await axe(container)).toHaveNoViolations()
})
```

### Mock API calls and test results

```
import React from 'react'
import {render, fireEvent, wait} from '@testing-library/react'
import {loadGreeting as mockLoadGreeting} from '../api'
import {GreetingLoader} from '../greeting-loader-01-mocking'

jest.mock('../api')

test('loads greetings on click', async () => {
  const testGreeting = 'TEST_GREETING'
  mockLoadGreeting.mockResolvedValueOnce({data: {greeting: testGreeting}})
  const {getByLabelText, getByText} = render(<GreetingLoader />)
  const nameInput = getByLabelText(/name/i)
  const loadButton = getByText(/load/i)
  nameInput.value = 'Mary'
  fireEvent.click(loadButton)
  expect(mockLoadGreeting).toHaveBeenCalledWith('Mary')
  expect(mockLoadGreeting).toHaveBeenCalledTimes(1)
  await wait(() =>
    expect(getByLabelText(/greeting/i)).toHaveTextContent(testGreeting),
  )
})
```

### Wait component changes

`findBy**** ` will wait to find the element on page, in the example is an error message after a form submission.

Example:
```
test('renders an error message from the server', async () => {
  const testError = 'test error'
  mockSavePost.mockRejectedValueOnce({data: {error: testError}})
  const fakeUser = userBuilder()
  const {getByText, findByRole} = render(<Editor user={fakeUser} />)
  const submitButton = getByText(/submit/i)

  fireEvent.click(submitButton)

  const postError = await findByRole('alert')
  expect(postError).toHaveTextContent(testError)
  expect(submitButton).not.toBeDisabled()
})
```

### Testing react-router

Using `createMemoryHistory` to create our own history and then use the router with the history we just created.

Example:
```
import React from 'react'
import {Router} from 'react-router-dom'
import {createMemoryHistory} from 'history'
import {render, fireEvent} from '@testing-library/react'
import {Main} from '../main'

test('main renders about and home and I can navigate to those pages', () => {
  const history = createMemoryHistory({initialEntries: ['/']})
  const {getByRole, getByText} = render(
    <Router history={history}>
      <Main />
    </Router>,
  )
  expect(getByRole('heading')).toHaveTextContent(/home/i)
  fireEvent.click(getByText(/about/i))
  expect(getByRole('heading')).toHaveTextContent(/about/i)
})
```

Refactored version of the example, with custom render function to wrap the component in the router.

```
mport React from 'react'
import {Router} from 'react-router-dom'
import {createMemoryHistory} from 'history'
import {render as rtlRender, fireEvent} from '@testing-library/react'
import {Main} from '../main'

// normally you'd put this logic in your test utility file so it can be used
// for all of your tests.
function render(
  ui,
  {
    route = '/',
    history = createMemoryHistory({initialEntries: [route]}),
    ...renderOptions
  } = {},
) {
  function Wrapper({children}) {
    return <Router history={history}>{children}</Router>
  }
  return {
    ...rtlRender(ui, {
      wrapper: Wrapper,
      ...renderOptions,
    }),
    // adding `history` to the returned utilities to allow us
    // to reference it in our tests (just try to avoid using
    // this to test implementation details).
    history,
  }
}

test('main renders about and home and I can navigate to those pages', () => {
  const {getByRole, getByText} = render(<Main />)
  expect(getByRole('heading')).toHaveTextContent(/home/i)
  fireEvent.click(getByText(/about/i))
  expect(getByRole('heading')).toHaveTextContent(/about/i)
  // you can use the `within` function to get queries for elements within the
  // about screen
})
```

### Testing custom hook

Testing custom hooks separately only if it is used in many components. If used only in one component it is enough to test the component that uses the hook.

Useful package to test hooks: [@testing-library/react-hooks](https://react-hooks-testing-library.com/usage/basic-hooks)

Example:

```
import {renderHook, act} from '@testing-library/react-hooks'
import {useCounter} from '../use-counter'

test('exposes the count and increment/decrement functions', () => {
  const {result} = renderHook(useCounter)
  expect(result.current.count).toBe(0)
  act(() => result.current.increment())
  expect(result.current.count).toBe(1)
  act(() => result.current.decrement())
  expect(result.current.count).toBe(0)
})

test('allows customization of the initial count', () => {
  const {result} = renderHook(useCounter, {initialProps: {initialCount: 3}})
  expect(result.current.count).toBe(3)
})
```

### Testing timers on unmount

Example:

```
import React from 'react'
import {render, act} from '@testing-library/react'
import {Countdown} from '../countdown'

beforeAll(() => {
  jest.spyOn(console, 'error').mockImplementation(() => {})
})

afterAll(() => {
  console.error.mockRestore()
})

afterEach(() => {
  jest.clearAllMocks()
  jest.useRealTimers()
})

test('does not attempt to set state when unmounted (to prevent memory leaks)', () => {
  jest.useFakeTimers()
  const {unmount} = render(<Countdown />)
  unmount()
  act(() => jest.runOnlyPendingTimers())
  expect(console.error).not.toHaveBeenCalled()
})
```

Where countdown is:

```
function Countdown() {
  const [remainingTime, setRemainingTime] = React.useState(10000)
  const end = React.useRef(new Date().getTime() + remainingTime)
  React.useEffect(() => {
    const interval = setInterval(() => {
      const newRemainingTime = end.current - new Date().getTime()
      if (newRemainingTime <= 0) {
        clearInterval(interval)
        setRemainingTime(0)
      } else {
        setRemainingTime(newRemainingTime)
      }
    })
    return () => clearInterval(interval)
  }, [])
  return remainingTime
}

export {Countdown}
```

### Snapshots

Inline snapsnots, simple usage example:

```
  expect(getByRole('alert').textContent).toMatchInlineSnapshot(
    `"There was a problem."`,
  )
```

## React integration tests

Testing a multipage form.
Example:

```
import React from 'react'
import {render, fireEvent} from '@testing-library/react'
import {submitForm as mockSubmitForm} from '../api'
import App from '../app'

jest.mock('../api')

test('Can fill out a form across multiple pages', async () => {
  mockSubmitForm.mockResolvedValueOnce({success: true})
  const testData = {food: 'test food', drink: 'test drink'}
  const {findByLabelText, findByText} = render(<App />)

  fireEvent.click(await findByText(/fill.*form/i))

  fireEvent.change(await findByLabelText(/food/i), {
    target: {value: testData.food},
  })
  fireEvent.click(await findByText(/next/i))

  fireEvent.change(await findByLabelText(/drink/i), {
    target: {value: testData.drink},
  })
  fireEvent.click(await findByText(/review/i))

  expect(await findByLabelText(/food/i)).toHaveTextContent(testData.food)
  expect(await findByLabelText(/drink/i)).toHaveTextContent(testData.drink)

  fireEvent.click(await findByText(/confirm/i, {selector: 'button'}))

  expect(mockSubmitForm).toHaveBeenCalledWith(testData)
  expect(mockSubmitForm).toHaveBeenCalledTimes(1)

  fireEvent.click(await findByText(/home/i))

  expect(await findByText(/welcome home/i)).toBeInTheDocument()
})
```

# e2e testing with cypress

Install cypress

`npm i -D cypress`

Configure eslint to work with cypress:

`npm i -D eslint-plugin-cypress`

Add `.eslintrc` in cypress folder:

```
{
  plugins: ['eslint-plugin-cypress'],
  extends: [ 'plugin:cypress/recommended'],
  env: {'cypress/globals': true},
}
```

Configure `cypress.json`
Example:
```
{
  "baseUrl": "http://localhost:8080",
  "integrationFolder": "cypress/e2e",
  "viewportHeight": 900,
  "viewportWidth": 400
}
```

Improve selectors with [cypress-testing-library](https://github.com/testing-library/cypress-testing-library)

## React Dev tools hooking
This is needed to hook cypress browser to our react app rather than the cypress react app.

`index.html`
```
 <script>
      if (window.Cypress) {
        window.__REACT_DEVTOOLS_GLOBAL_HOOK__ =
          window.parent.__REACT_DEVTOOLS_GLOBAL_HOOK__
      }
    </script>
```
## Run cypress

`npm i -D start-server-and-test is-ci`

In `package.json`

```
"cy:run": "cypress run",
"cy:open": "cypress open",
"test:e2e": "is-ci \"test:e2e:run\" \"test:e2e:dev\"",
"pretest:e2e:run": "npm run build",
"test:e2e:run": "start-server-and-test start http://localhost:8080 cy:run",
"test:e2e:dev": "start-server-and-test dev http://localhost:8080 cy:open",
```

## Cypress tests

### Mock server response and check for errors

Example:
```
  it(`should show an error message if there's an error registering`, () => {
    cy.server()
    cy.route({
      method: 'POST',
      url: 'http://localhost:3000/register',
      status: 500,
      response: {},
    })
    cy.visit('/register')
      .findByText(/submit/i)
      .click()
      .findByText(/error.*try again/i)
  })
})
```

## Node backend testing

### Test many cases
`jest-in-case` helps to case many cases on a pure function.

Example how to test including also some of the test inputs in the titles:

```

// Testing Pure Functions
// 💯 improved titles for jest-in-case

import cases from 'jest-in-case'
import {isPasswordAllowed} from '../auth'

function casify(obj) {
  return Object.entries(obj).map(([name, password]) => ({
    name: `${password} - ${name}`,
    password,
  }))
}

cases(
  'isPasswordAllowed: valid passwords',
  ({password}) => {
    expect(isPasswordAllowed(password)).toBe(true)
  },
  casify({'valid password': '!aBc123'}),
)

cases(
  'isPasswordAllowed: invalid passwords',
  ({password}) => {
    expect(isPasswordAllowed(password)).toBe(false)
  },
  casify({
    'too short': 'a2c!',
    'no letters': '123456!',
    'no numbers': 'ABCdef!',
    'no uppercase letters': 'abc123!',
    'no lowercase letters': 'ABC123!',
    'no non-alphanumeric characters': 'ABCdef123',
  }),
)
```

The `Object.entries()` method returns an array of a given object's own enumerable string-keyed property `[key, value]` pairs.


### Testing error message with .toMatchInlineSnapshot


 Example:
 ```
 test('createListItem returns a 400 error if no bookId is provided', async () => {
  const req = buildReq()
  const res = buildRes()

  await listItemsController.createListItem(req, res)

  expect(res.status).toHaveBeenCalledWith(400)
  expect(res.status).toHaveBeenCalledTimes(1)
  expect(res.json.mock.calls[0]).toMatchInlineSnapshot(`
    Array [
      Object {
        "message": "No bookId provided",
      },
    ]
  `)
  expect(res.json).toHaveBeenCalledTimes(1)
})
```
`toMatchInlineSnapshot()` adds the snapshots inline on the first run of the test. This is useful with error messages because it is easy to update the tests if we change the error message. Avoiding copying and pasting manually.

### Check function calls

`.toHaveBeenCalledWith`
`.toHaveBeenCalledTimes`

### Integration test on backend

Start server before the tests, stop it at the end.
Clear the database between tests.

```
let server

beforeAll(async () => {
  server = await startServer({port: 8000})
})

afterAll(() => server.close())

beforeEach(() => resetDb())
```

#### Randomize port

`jest.config.js`

```
  setupFilesAfterEnv: [require.resolve('./setup-env')],
```

#### Setup function
To for example insert a test user in the database

For example:


```
async function setup() {
  const testUser = await insertTestUser()
  const authAPI = axios.create({baseURL})
  authAPI.defaults.headers.common.authorization = `Bearer ${testUser.token}`
  authAPI.interceptors.response.use(getData, handleRequestFailure)
  return {testUser, authAPI}
}
```



Where `setup-env.js`

```
const port = 8800 + Number(process.env.JEST_WORKER_ID)
process.env.PORT = process.env.PORT || port
```

### Generic expect on strings

Example:

```
expect(response.data.user).toEqual({
    token: expect.any(String),
    id: expect.any(String),
    username,
  })
```




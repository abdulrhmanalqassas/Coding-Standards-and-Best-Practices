# Coding Standards and Best Practices

Coding style guidelines for the frontend team

## Table of Contents

1. [Introduction](#overview)
2. [Variables](#variables)
3. [Functions](#functions)
4. [Testing](#testing)
5. [Team Agreements and Decisions](#agreements)

## Overview

Our guidelines, principles, and standards help us:

- Maintain high-quality code consistently across all projects.
- Allow several developers to work on the same codebase simultaneously in a unified manner.
- Produce code that's more reliable and easier to understand, debug, and maintain.
- Support the creation of reusable code.

## Contributing

We prioritize **consistent coding practices** over **individual coding styles**. This document will be updated regularly
to keep up with the needs of our development team and the nature of our projects.

## **Variables**

### Use meaningful and pronounceable variable names

**Bad:**

```javascript
const d = new Date();
```

**Good:**

```javascript
const currentDate = new Date();
```

### Avoid using magic numbers or magic strings

**Bad:**

```javascript
// What is 86400000 for?
setTimeout(someFunction, 86400000);
```

**Good:**

```javascript
// Declare them as capitalized named constants.
const MILLISECONDS_PER_DAY = 60 * 60 * 24 * 1000; //86400000;

setTimeout(someFunction, MILLISECONDS_PER_DAY);
```

### Use explanatory variables

**Bad:**

```javascript
const address = "city, country 95014";
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/;
saveCityZipCode(
  address.match(cityZipCodeRegex)[1],
  address.match(cityZipCodeRegex)[2]
);
```

**Good:**

```javascript
const address = "city, country 95014";
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/;
const [_, city, zipCode] = address.match(cityZipCodeRegex) || [];
saveCityZipCode(city, zipCode);
```

### Avoid Mental Mapping

**Bad:**

```javascript
const locations = ["Austin", "New York", "San Francisco"];
locations.forEach((l) => {
  doStuff();
  doSomeOtherStuff();
  // Wait, what is `l` for again?
  dispatch(l);
});
```

**Good:**

```javascript
const locations = ["Austin", "New York", "San Francisco"];
locations.forEach((location) => {
  doStuff();
  doSomeOtherStuff();
  dispatch(location);
});
```

### Don't add unneeded context

If your class/object name tells you something, don't repeat that in your
variable name.

**Bad:**

```javascript
const Car = {
  carMake: "Mercedes",
  carModel: "GLC",
  carColor: "Sliver",
};

function paintCar(car, color) {
  car.carColor = color;
}
```

**Good:**

```javascript
const Car = {
  make: "Mercedes",
  model: "GLC",
  color: "Sliver",
};

function paintCar(car, color) {
  car.color = color;
}
```

### Use default parameters instead of short circuiting or conditionals

Default parameters are often cleaner than short circuiting. Be aware that if you
use them, your function will only provide default values for `undefined`
arguments. Other "falsy" values such as `''`, `""`, `false`, `null`, `0`, and
`NaN`, will not be replaced by a default value.

**Bad:**

```javascript
function generateNewName(name) {
  const newName = name || "Hipster Brew Co.";
  // ...
}
```

**Good:**

```javascript
function generateNewName(name = "Hipster Brew Co.") {
  // ...
}
```

## **Functions**

### Function arguments (2 or fewer ideally)

One or two arguments is the ideal case, and three should be avoided if possible.
Anything more than that means your function is trying to do too much and should be consolidated. Most of the time a
higher-level object will suffice as an
argument.

**Bad:**

```javascript
function createMenu(title, body, buttonText, cancellable) {
  // ...
}

createMenu("Form Title", "body text", "Save", true);
```

**Good:**

```javascript
function createMenu({ title, body, buttonText, cancellable }) {
  // ...
}

createMenu({
  title: "Form Title",
  body: "body text",
  buttonText: "Save",
  cancellable: true,
});
```

### Functions should do one thing

When functions do more than one thing, they are harder to compose, test, and reason about.

**Bad:**

```javascript
function emailClients(clients) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

**Good:**

```javascript
function emailActiveClients(clients) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

### Function names should say what they do

**Bad:**

```javascript
function addToDate(date, month) {
  // ...
}

const date = new Date();

// It's hard to tell from the function name what is added
addToDate(date, 1);
```

**Good:**

```javascript
function addMonthToDate(month, date) {
  // ...
}

const date = new Date();
addMonthToDate(1, date);
```

### Functions should only be one level of abstraction

When you have more than one level of abstraction your function is usually
doing too much. Splitting up functions leads to reusability and easier
testing.

**Bad:**

```javascript
function parseBetterJSAlternative(code) {
  const REGEXES = [
    // ...
  ];

  const statements = code.split(" ");
  const tokens = [];
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      // ...
    });
  });

  const ast = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });
}
```

**Good:**

```javascript
function parseBetterJSAlternative(code) {
  const tokens = tokenize(code);
  const syntaxTree = parse(tokens);
  syntaxTree.forEach((node) => {
    // parse...
  });
}

function tokenize(code) {
  const REGEXES = [
    // ...
  ];

  const statements = code.split(" ");
  const tokens = [];
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      tokens.push(/* ... */);
    });
  });

  return tokens;
}

function parse(tokens) {
  const syntaxTree = [];
  tokens.forEach((token) => {
    syntaxTree.push(/* ... */);
  });

  return syntaxTree;
}
```

### Don't use flags as function parameters

Flags tell your user that this function does more than one thing. Functions should do one thing. Split out your
functions if they are following different code paths based on a boolean.

**Bad:**

```javascript
function createFile(name, temp) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

**Good:**

```javascript
function createFile(name) {
  fs.create(name);
}

function createTempFile(name) {
  createFile(`./temp/${name}`);
}
```

### Use Pure Functions Wherever Possible

**Objective:** Strive to write functions that are pure. A pure function is one that:

- Has No Side Effects: It does not alter any external state (e.g., global variables, disk storage) nor does it depend on
  any external state that is subject to change.
- Returns Consistent Outputs: For the same inputs, it always returns the same outputs, making its behavior predictable
  and testable.

**Bad:**

```javascript
// Bad practice: Impure function with external dependency
let taxRate = 0.07; // External state

function calculateTotalWithTax(prices) {
  return prices.reduce((total, price) => total + price * (1 + taxRate), 0);
}
```

**Good:**

```javascript
// Good practice: Pure function to calculate the total
function calculateTotal(prices, taxRate) {
  return prices.reduce((total, price) => total + price * (1 + taxRate), 0);
}
```

### Encapsulate conditionals

**Bad:**

```javascript
if (state === "fetching" && isEmpty(listNode)) {
  // ...
}
```

**Good:**

```javascript
function shouldShowSpinner(state, listNode) {
  return state === "fetching" && isEmpty(listNode);
}

if (shouldShowSpinner(state, listNodeInstance)) {
  // ...
}
```

### Avoid negative conditionals

**Bad:**

```javascript
function isDOMNodeNotPresent(node) {
  // ...
}

if (!isDOMNodeNotPresent(node)) {
  // ...
}
```

**Good:**

```javascript
function isDOMNodePresent(node) {
  // ...
}

if (isDOMNodePresent(node)) {
  // ...
}
```

## **Testing**

### Single concept per test

**Bad:**

```javascript
import { format, addDays, parse } from "date-fns";

describe("date-fns", () => {
  it("handles date boundaries", () => {
    let date;

    date = parse("1/1/2024", "M/d/yyyy", new Date());
    date = addDays(date, 30);
    expect(format(date, "M/d/yyyy")).toEqual("1/31/2024");

    date = parse("2/1/2024", "M/d/yyyy", new Date());
    date = addDays(date, 28);
    expect(format(date, "M/d/yyyy")).toEqual("2/29/2024");

    date = parse("2/1/2024", "M/d/yyyy", new Date());
    date = addDays(date, 28);
    expect(format(date, "M/d/yyyy")).toEqual("3/1/2024");
  });
});
```

**Good:**

```javascript
import { format, addDays, parse } from "date-fns";

describe("date-fns", () => {
  it("handles 30-day months", () => {
    let date = parse("1/1/2024", "M/d/yyyy", new Date());
    date = addDays(date, 30);
    expect(format(date, "M/d/yyyy")).toEqual("1/31/2024");
  });

  it("handles leap year", () => {
    let date = parse("2/1/2024", "M/d/yyyy", new Date());
    date = addDays(date, 28);
    expect(format(date, "M/d/yyyy")).toEqual("2/29/2024");
  });

  it("handles non-leap year", () => {
    let date = parse("2/1/2024", "M/d/yyyy", new Date());
    date = addDays(date, 28);
    expect(format(date, "M/d/yyyy")).toEqual("3/1/2024");
  });
});
```

### Accessible Queries

Consider mimicking the user for more realistic interaction simulation and future proofness ..

**Fine:**

```javascript
// Instead of
const element = screen.getByTestId("element-id");
```

**Better:**

```javascript
// Use
const element = screen.getByRole("textbox", { name: /element label/i });
```

### Mock Data Management

If our mock data expands, consider moving it to a dedicated file for better organization.

```javascript
// mockData.js
export const mockPickup = {
  formatted_address: "P. Sherman, 42 Wallaby Way, Sydney",
};

export const mockDestination = {
  formatted_address: "221B Baker Street",
};
```

### Dynamic Behavior Tests

Add tests for dynamic behaviors like conditional rendering.

```javascript
it("renders alternative text when address is not provided", () => {
  render(<KidnapNemo pickup={{}} destination={{}} />);
  expect(screen.getByText("No address provided")).toBeInTheDocument();
});
```

### Error Handling

Ensure error states or edge cases are managed gracefully.

```javascript
it("displays error message on incomplete data", () => {
  render(<RescueNemo pickup={null} destination={null} />);
  expect(screen.getByText("Error loading addresses")).toBeInTheDocument();
});
```

### Reducing Repetition

Use **beforeEach** for setup steps that are common across multiple tests.

```javascript
beforeEach(() => {
  render(<MyComponent pickup={mockPickup} destination={mockDestination} />);
});
```

### Context Integration

Test how the component behaves under different context providers.

```javascript
it("renders correctly under different contexts", () => {
  render(
    <UserContext.Provider value={mockUser}>
      <MyComponent pickup={mockPickup} destination={mockDestination} />
    </UserContext.Provider>
  );
  expect(screen.getByText(mockUser.name)).toBeInTheDocument();
});
```

### Performance Assertions

Monitor performance to ensure it remains within acceptable boundaries.

```javascript
it('renders efficiently', () => {
    const { container } = render(<FindNemo... />);
    expect(performance.now() - startTime).toBeLessThan(200);
});
```

### Accessibility Checks

Utilize tools like jest-axe to ensure accessibility compliance.

```javascript
import { axe } from 'jest-axe';

it('is accessible', async () => {
    const { container } = render(<FindNemo... />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
});

```

### Modular Test Design

Break tests into smaller, more specific cases to avoid monolithic test functions and enhance clarity.

```javascript
it("should display the default image when no image is provided", () => {
  render(<ImageComponent />);
  expect(screen.getByRole("img")).toHaveAttribute("src", "default-image.png");
});
```

### Parameterized Tests

Use parameterized tests for scenarios where you need to run the same test logic with different data. This reduces redundancy and increases the comprehensiveness of your tests.

```javascript
const inputs = [
  { input: "data1", expected: "result1" },
  { input: "data2", expected: "result2" },
];
inputs.forEach(({ input, expected }) => {
  it(`properly handles ${input}`, () => {
    expect(processInput(input)).toBe(expected);
  });
});
```

### Get enough coverage for being confident, ~80% seems to be the lucky number

The purpose of testing is to get enough confidence for moving fast, obviously the more code is tested the more confident
the team can be:

- 10–30% is obviously too low to get any sense about the build correctness
- 100% is very expensive and might shift your focus from the critical paths to the exotic corners of the code
- 70–90% is a good balance between the confidence and the cost

## Agreements

### Team Agreements and Decisions

Below is a table of the agreements and decisions that the team has already made:

| Decision                   | Details                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------- |
| Package Manager            | npm                                                                                   |
| Linting                    | Eslint                                                                                |
| Formatter                  | Prettier                                                                              |
| Styling                    |            sacss                                                                      |
| Component Library          | [chakra-ui](https://chakra-ui.com/)                                                   |
| Pull Request Template      | [This template](./Resources/pull_request_template.md)                                 |
| Git Ignore                 | [This template](./Resources/.gitignore)                                               |
| Commit Message             | Use the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format |
| Branch Naming Convention   | `<ticket_number>-<ticket_name>` Example: `CMB-123-add-login-button`                   |
| Code Review                | At least one team member must approve the PR before merging                           |
| Code Quality Tool          | SonarQube                                                                             |
| Error Tracking and Logging | Sentry                                                                                |
| i18n                       | i18n                                                                                  |
| Testing Framework          | Jest                                                                                  |
| Form Handling              | [React Hook Form](https://react-hook-form.com/)                                       |
| State Management           | [Redux Toolkit](https://redux-toolkit.js.org/)                                        |
| Data Cache                 | [React Query](https://react-query.tanstack.com/)                                      |
| Mocking Library            | [MSW](https://mswjs.io/)                                                              |
| Data Validation            | [Zod](https://zod.dev/)                                                               |
| API Client                 | [Axios](https://axios-http.com/)                                                      |
| Date and Time Handling     | [date-fns](https://date-fns.org/) (In case we need advanced date handling only)       |

## References

Some suggested libraries and tools to use in the projects:

| Purpose       | Details                                         |
| ------------- | ----------------------------------------------- |
| Drag and Drop | [Swapy](https://swapy.tahazsh.com/)             |
| Charts        | [Chart.js](https://www.chartjs.org/)            |
| Onboarding    | [Driver.js](https://driverjs.com/)              |
| Toasts        | [React Hot Toast](https://react-hot-toast.com/) |
| React Hooks   | [useHooks](https://usehooks.com/)               |

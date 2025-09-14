# Coding Standards and Best Practices

Coding style guidelines for the frontend team

### Table of Contents

1. [Introduction](#overview)  
2. [Variables](#variables)  
3. [Functions](#functions)  
4. [Testing](#testing)  
5. [Chakra UI](#chakra-ui)  
6. [Redux](#redux)  
7. [Team Agreements and Decisions](#agreements)  
8. [References](#references)

## Overview

Our guidelines, principles, and standards help us:

- Maintain high-quality code consistently across all projects.  
- Allow multiple developers to work on the same codebase in a unified manner.  
- Produce code that's more reliable and easier to understand, debug, and maintain.  
- Support the creation of reusable code.

## Contributing

We prioritize **consistent coding practices** over **individual coding styles**. This document will be updated regularly to keep up with the needs of our development team and the nature of our projects.

## Variables

### Use meaningful and pronounceable variable names

**Bad:**

```javascript
const d = new Date();
````

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
const MILLISECONDS_PER_DAY = 60 * 60 * 24 * 1000; // 86400000

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

If your class/object name tells you something, don't repeat that in your variable name.

**Bad:**

```javascript
const Car = {
  carMake: "Mercedes",
  carModel: "GLC",
  carColor: "Silver",
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
  color: "Silver",
};

function paintCar(car, color) {
  car.color = color;
}
```

### Use default parameters instead of short circuiting or conditionals

Default parameters are often cleaner than short circuiting. Be aware that if you use them, your function will only provide default values for `undefined` arguments. Other “falsy” values such as `''`, `false`, `null`, `0`, and `NaN` will *not* be replaced by a default value.

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

## Functions

### Function arguments (2 or fewer ideally)

One or two arguments is the ideal case; three should be avoided if possible. Anything more than that means your function is likely doing too much. Most of the time a higher-level object will suffice as an argument.

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

When you have more than one level of abstraction your function is usually doing too much. Splitting up functions leads to reusability and easier testing.

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

Flags tell your user that this function does more than one thing. Functions should do one thing. Split out your functions if they are following different code paths based on a boolean.

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

* Has No Side Effects: It does not alter any external state (e.g., global variables, disk storage) nor does it depend on any external state that is subject to change.
* Returns Consistent Outputs: For the same inputs, it always returns the same outputs, making its behavior predictable and testable.

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

## Testing

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


### Error Handling

Ensure error states or edge cases are managed gracefully.

```javascript
it("displays error message on incomplete data", () => {
  render(<RescueNemo pickup={null} destination={null} />);
  expect(screen.getByText("Error loading addresses")).toBeInTheDocument();
});
```


## Chakra UI

### Principles & Design Patterns

* Style Props: Use style props instead of writing separate CSS or using too many `styled()` wrappers. This maintains consistency and makes theme overrides easier. ([v2.chakra-ui.com][1])
* Simplicity: Keep component APIs simple; each component should do what its name suggests and expose minimal props. ([v2.chakra-ui.com][1])
* Composition: Build complex UI by composing smaller, focused Chakra components (e.g. `Box`, `Flex`, `Stack`, etc.). Don’t make one component do too much. ([Daily.dev][2])
* Accessibility: Always consider ARIA attributes, keyboard navigation, focus management, color contrast, etc. Chakra has built-in support but correct usage is the developer’s responsibility. ([Daily.dev][2])
* Dark Mode Support: Components should work correctly in both light and dark mode. Use `useColorMode`, theme tokens, and ensure contrasts are acceptable. ([v2.chakra-ui.com][1])

### Theming & Responsiveness

* Use `extendTheme` to customize design tokens (colors, spacing, typography, breakpoints) in one place. ([Daily.dev][2])
* Value responsiveness via Chakra’s array/object syntax for style props (for example for widths, paddings, display etc.). Test at different breakpoints. ([Daily.dev][2])
* Reuse style tokens (colors, spacing etc.) instead of duplicated hardcoded values.
* Avoid uncontrolled growth of inline style objects that force re-renders; prefer theme styles and memoization when needed.

### Component Structure & Naming

* Boolean props should be named with auxiliary verbs such as `isSomething`, `hasSomething`, `shouldSomething`. For example: `isLoading`, `isDisabled`. ([v2.chakra-ui.com][1])
* Co-locate component logic and styling in relevant places; separate presentational components vs container logic.
* Keep components small and focused. If you find many props, consider splitting.

## Redux

### Core Principles (from Redux Official Style Guide)

* **State immutability**: Do not mutate state directly. Use immutable patterns or tools like Immer. ([redux.js.org][3])
* **Pure reducers**: Reducer functions must not have side effects (API calls, timeouts, randomness, etc.). ([redux.js.org][3])
* **Only serializable values** in state and actions: avoid including things like Promises, class instances, functions, non-serializable data. ([redux.js.org][3])
* Prefer having a **single Redux store per app**. ([redux.js.org][3])

### Organizing Redux Code

* Use **Redux Toolkit** (RTK) as the standard approach for creating slices, store, and middleware, because it encodes many best practices. ([redux.js.org][3])
* Structure code by **feature folders**: each “feature” (slice) should include its actions, reducers, selectors, types, maybe tests. This avoids spread-out code. ([redux.js.org][3])
* Co-locate selectors with reducers, or in the same slice file to reduce friction and make state shape more understandable. ([Medium][4])

### Derived State, Selectors & Memoization

* Derived data (data that can be computed from existing state) should be computed via selectors, not stored redundantly in the store. ([Medium][4])
* Use memoization (e.g. `reselect`) for selectors that are expensive or run frequently. ([Medium][4])

### Handling Side Effects & Async Logic

* Use middleware (thunks, sagas, or RTK Query) for asynchronous operations, side effects, API interactions. Reducers stay pure. ([redux.js.org][3])
* Maintain explicit state for loading, success, error states in async flows so UI can reflect them.

### Testing & Debugging Redux

* Write tests for reducers, selectors, and any async logic or action creators where logic isn’t trivial.
* Use DevTools, logging, or similar tools to inspect actions and state transitions.
* Ensure same input + action always produce same output (pure functions), to facilitate predictability and debugging.

## Team Agreements and Decisions

Below is a table of the agreements and decisions that the team has already made:

| Decision                   | Details                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------- |
| Package Manager            | npm                                                                                   |
| Linting                    | Eslint                                                                                |
| Formatter                  | Prettier                                                                              |
| Styling                    | sacss                                                                                 |
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

* Redux Style Guide — Official best practices for Redux emphasis on immutability, pure reducers, etc. ([redux.js.org][3])
* Chakra UI Design Principles & Docs — Information about theming, style props, responsiveness, composition, dark mode. ([v2.chakra-ui.com][1])
* Chakra UI Adoption Guide — key features and trade-offs when using Chakra UI in production. ([LogRocket Blog][5])
* Redux Best Practices (community articles) — e.g. rules about selectors, derived data, normalizing state. ([Medium][4])
* Chakra UI Design Patterns — examples of modular design, layout, responsiveness. ([Daily.dev][2])


[1]: https://v2.chakra-ui.com/getting-started/principles?utm_source=chatgpt.com "Design Principles - Chakra UI"
[2]: https://daily.dev/blog/chakra-ui-design-patterns-basics?utm_source=chatgpt.com "Chakra UI Design Patterns: Basics - Daily.dev"
[3]: https://redux.js.org/style-guide/?utm_source=chatgpt.com "Redux Style Guide"
[4]: https://medium.com/%40kylpo/redux-best-practices-eef55a20cc72?utm_source=chatgpt.com "Redux Best Practices - Medium"
[5]: https://blog.logrocket.com/chakra-ui-adoption-guide/?utm_source=chatgpt.com "Chakra UI adoption guide: Overview, examples, and alternatives"

# Principles

## Purpose of Automated Testing

see [The True Purpose of Testing](https://www.epicweb.dev/the-true-purpose-of-testing)

## 01. Intention

We have to evaluate the intention of the code we want to test. Sometimes the intention closely maps to the
implementation. But quite often, the implementation is complex, and it may be challenging to see the intention.

A test should never test the implementation. Instead, the test should validate that the intention of the code was
achieved. A test is only useful when it validates the code's intention.

## Why Write Tests?

Testing seems complicated, but the concept is straightforward. Taking the simple example of a function that outputs
some text when called, I can test it manually by calling it and confirming that the correct text was output. Or, I
can write a test to assert that the correct text was output. If I or someone else changes the implementation code of
the function, then I can test it again to confirm the correct text is still being output. I don't want to do this
manually. In most programs, there will be many functions that all work together to achieve a desired result. Having
to manually test each function every time, something changes in the code would be challenging and prone to human error.
Having an automated test makes refactoring easier and gives me confidence that my changes did not break anything.

## The Most Basic Example of an Automated Test

```Typescript
// helpers.ts

export function greet(name: string): string {
    return `Hello, ${name}`;
}
```

```Typescript
// helpers.test.ts

import { greet } from 'helpers'

const message = greet('Ian');

if (message !== 'Hello, Ian')
    throw new Error(`Test greet failed. Expected 'Hello, Ian' but recieved ${ message }.`);
else
    console.log('Test greet passed');
```

This test can be run with the following command.

```Bash
npx tsx helpers.test.ts
```

Actually to fully automate things, the tsx command can be run in watch mode while refactoring, so it will rerun
automatically every time I save changes.

```Bash
npx tsx helpers.test.ts
```

[tsx](https://tsx.is/) is a popular npm package/command to run Typescript in Node.js

## 02. Implementation Details

We write tests to fail. In TDD we write the test first and then write implementation code until the test passes.
Whether writing the test first or second, it should still fail if the intention is not met. The test is not
concerned with implementation details, but if those details change and cause the intent not to be met, then we want
the test to fail.

Always trust the intention. If "John" comes along and changes our code to

```Typescript
export function greet(name: string): string {
    return `Howdy, ${name}`;
}
```

Then the test will fail. By comparing the intention defined in the test with the implementation in the function, we
can see that the problem here is in the implementation. Don't change the test if you are confident that the
intention is correct.

> The implementation may change, but the intention stays the same.

The case may be that John is the CEO and he wants the intention to be changed but never assume, always ask, and
always trust the test first.

[The Golden Rule of Assertions](https://www.epicweb.dev/the-golden-rule-of-assertions) is an article by Artem that
goes into more detail about testing only the implementation and gives a couple of examples of how to remove
implementation details from your tests.

## Section Elaboration

I learned how to think about testing to write better tests by learning the following concepts:

1. What is the purpose of tests? The purpose of tests is to validate the intention by failing if and only if
   the intention is not met. It is not a tests concern, how the intention is achieved but only that it is achieved.
2. Intention vs. Implementation. Intention is what I expect the code to do. Implementation is any detail about how
   the system does it.
3. What to ask myself when writing a test. What is the intention I am testing here? When will my test fail? Is any
   part of my test relying on an implementation detail, and if so, how can I remove it from my test to ensure that I
   am only testing the intention?

Two Great companion articles for this section:
[The True Purpose of Testing](https://www.epicweb.dev/the-true-purpose-of-testing)
[The Golden Rule of Assertions](https://www.epicweb.dev/the-golden-rule-of-assertions)

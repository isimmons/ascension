# Test Structure

Every test consists of 3 phases.

1. Setup (optional)—If there is anything that needs to be set up to perform the test (more on this later).
2. Action—Run the code and get a result
3. Assert—Compare the result with the expected result and throw an error if the result is not correct

[Anatomy of a Test](https://www.epicweb.dev/anatomy-of-a-test)

## Setup

The setup step can be simple or complex depending on the needs for the test. In the setup step, you prepare
everything around your code so the code will actually run. Establishing test environment and definition,
putting the code into a desired initial state, handling unwanted side effects, mocking, and establishing test
boundaries are some of the things you take care of in the setup step.

When using a testing framework, some of this work is done for you.

Some things you might see in a setup step include:

- running the main app or rendering an isolated component
- mocking network requests
- using dependency injection
- initiating test database connections
- wrapping test code in providers

TODO: create separate doc for this video. Excellent resource.

[Dissecting Complexity in Tests](https://youtu.be/nUozijHdgMM?si=WCd3DyE5zSDWpgGV)

## Action

In the action step, perform actions on the test code. Actions are determined by the expectations and how to bring
the code to the expected state. A simple example would be storing the result of calling a sum function that returns
the sum of two numbers. Then in the "assert" step, I can assert that the result matches my expectation, therefore
validating the intent. The intent here is that given two numbers, the sum function will return the correct sum of
those two numbers.

Actions should resemble the way an end user would actually use the code.

An end user interacting with the code through a web browser would probably call for an integration test where the UI
is tested. For example, a form with two number inputs and a "submit" button. The intent would be when the user enters
two numbers and clicks the button, the UI displays the correct sum of the two numbers.

A programmer, using your library/framework to build a user interface is another type of user. The intent and the
test might be a unit test of the sum function where there is no real setup. As the author of the framework, our
concern here is the programmer as our end user.

Some actions you may make in a test are:

- call functions and methods
- navigate to a particular page
- interact with DOM (click events and the results of the listeners)
- dispatch network requests

## Assertion

In the assertion step, you compare the results from the action step to the expected result to validate the
intent.

Assertions are the most important part of the test. They must throw and throw reliably and for the right reason. If
they don't, then the entire test is useless.

It may not always be clear what and to which extent to assert. Remember the
[Golden Rule of Assertions](https://www.epicweb.dev/the-golden-rule-of-assertions)

## Imperative vs Declarative

A testing framework provides us with declarative methods such as "expect" and "toBe" which make it much easier to
structure our tests in a clean and readable way. In this lesson, we create our own version of this.

Before adding the expect and toBe methods, our test has imperative code in the assertion. This means our code is too
concerned with how to perform the assertion.

We can write our test in a declarative way by using the new expect and toBe methods.

```Typescript
// the functions under test
function greet(name: string) {
	return `Hello, ${name}!`
}

function congratulate(name: string) {
	return `Congrats, ${name}!`
}


// the actual test
expect(greet('John')).toBe('Hello, John!')

expect(congratulate('Sarah')).toBe('Congrats, Sarah!')


// the test helper functions
function expect(actualValue: unknown) {
	return {
		toBe: (expectedValue: unknown) => {
			if (actualValue !== expectedValue)
				throw new Error(`Expected ${actualValue} to equal to ${expectedValue}`)

			console.log('Tests Passed...')
		},
	}
}
```

In this simple example, there is no need for the setup step. Also, notice the action and assert steps are combined
into oneliners. Some might argue to never do this, but in simple cases, it is OK. If the test grows in complexity, it
will be easy enough to split into separate action and assert lines later, but typically, a test does not change
unless business logic dictates a change in implementation that requires it.

## Test Blocks

We can take things further by using test blocks. Vitest and other testing frameworks provide functions such as
"describe," "test," and "it." The "it" method just makes it easier to use declarative language because it makes your
test look
more like a sentence. `it('does this or that', () => {//test code...})`. The 'test' method works the same way.
Using test blocks helps us to separate tests visually, write more declarative titles for them, and handle the
running and pass or fail output for each one separately.

```Typescript
function greet(name: string) {
	return `Hello, ${name}!`
}

function congratulate(name: string) {
	return `Congrats, ${name}!`
}

it('returns a greeting message for the given name', () => {
	expect(greet('John')).toBe('Hello, John!')
})

it('returns a congratulations message for the given name', () => {
	expect(congratulate('Sarah')).toBe('Congrats, Sarah!')
})

function expect(actual: unknown) {
	return {
		toBe(expected: unknown) {
			if (actual !== expected) {
				throw new Error(`Expected ${actual} to equal to ${expected}`)
			}
		},
	}
}

function it(title: string, callback: () => void) {
	try {
		callback()

		console.log(`✓ ${title}`)
	} catch (error: unknown) {
		console.error(`✗ ${title}`)
		if (error instanceof Error) console.error(error.message)
		else console.error(error)
	}
}
```

This way of separating tests and making them more declarative is a great help when there is an error. If all we had
to go on was the conditionally thrown error from before, we would have a challenging time tracking down exactly where
the error is.

## Test Files

What we have done up to this point is called "in source testing." It can have its place, but for the most part it is
undesirable. A developer is limited to using only named imports because if they import "*" from greet.ts, they will
get not only the greet and congratulate functions but also the tests. Besides, it makes it hard to find and run
tests if they are scattered around in the source code.

Test files allow us to separate our implementation code physically from the test code. Test runners provide a way to
identify tests by directory name and/or file name. Examples are "__tests__" for a directory name or "func.test.ts"
or "func.spec.ts."

```Typescript
    // greet.test.ts
    import { greet, congratulate } from './greet.js'

    it('returns a greeting message for the given name', () => {
        expect(greet('John')).toBe('Hello, John!')
    })

    it('returns a congratulation message for the given name', () => {
        expect(congratulate('Sarah')).toBe('Congrats, Sarah!')
    })
```

To enable this type of test separation in our example, I did the following.

1. Move greet and congratulate to their own file (greet.ts)
2. Move the tests to greet.test.ts and import the greet and congratulate functions
3. Move expect, test, and it to a setup.ts file and make them globally available. We could just export and import
   them into the test file, but where's the fun in that?

    ```Typescript
        // setup.ts
        function expect(actual: unknown) {
            return {
                toBe(expected: unknown) {
                    if (actual !== expected) {
                        throw new Error(`Expected ${actual} to equal to ${expected}`)
                    }
                },
            }
        }

        function test(title: string, callback: () => void) {
            try {
                callback()
                console.log(`✓ ${title}`)
            } catch (error: unknown) {
                console.error(`✗ ${title}`)
                if (error instanceof Error) console.error(error.message, '\n')
                else console.error(error, '\n')
            }
        }

        // make globally available to node
        globalThis.expect = expect
        globalThis.test = test
        globalThis.it = test
    ```

4. Make Typescript happy by declaring types, or you will suffer the red squigglies in the test file. There are a couple
   of
   ways to do this.

    - A. Use a global.d.ts file

        ```Typescript
          interface Assertion {
            [K: string]: (expected: unknown) => void
          }

          declare function expect(actual: unknown): Assertion
          declare function test(title: string, callback: () => void): void
          declare function it(title: string, callback: () => void): ReturnType<test>
        ```

    - B. Top of the setup.ts file

        ```Typescript
          interface Assertion {
            [K: string]: (expected: unknown) => void
          }

          declare global {
            var expect: (actual: unknown) => Assertion
            var test: (title: string, callback: () => void) => void
            var it: typeof test
          }
        ```

5. Use the "--import" flag to import setup.ts when running the tests

   ```Bash
    npx tsx watch --import ./setup.ts  greet.test.ts
   ```

> [!NOTE]
>
> With either of the above methods for declaring types and globals, we can still export our functions and import
> them explicitly in our test file if we wanted to for some reason.

## Hooks

In our example, the intention has changed thanks to Peter the Project Manager. Now we are adding the current date
into the greeting. So, we change the test to test for the new intention. As is, the test only adds 'Monday' to the
greeting, so it will only pass if we run it on Monday.

We need to mock the date function, so it will always return 'Monday' when we run the function in the action part of
our test. To do this, we will make use of "beforeAll" and "afterAll" hooks.

Test runners come with these hooks, so we can perform setup before all tests and cleanup after all tests. This is
typically where we initiate the testing database or mock something tests, and then we make sure to clean up in the
afterAll hook by undoing actions performed in the beforeAll hook. It is important to clean up in the afterAll hook so
that any side effects we deal with in the beforeAll hook will not leak into other tests in other files. Right now we
only have one file, but as we test more or our app, we will be adding more files.

To make this work, first we create the global beforeAll and afterAll hooks in setup.ts and declare their types.

```Typescript
declare global {
	var expect: (actual: unknown) => Assertions
	var test: (title: string, callback: () => void) => void
	var beforeAll: (callback: () => void) => void
	var afterAll: (callback: () => void) => void
}

// other functions...

globalThis.beforeAll = function (callback) {
	callback()
}

globalThis.afterAll = function (callback) {
	process.on('beforeExit', callback)
}
```

The beforeAll function runs a callback when we call it at the top of our test file. The afterAll function does the
same, but only after the beforeExit event is fired from the node process. This event is fired right before the node
process exits, so it will run the callback after all tests in the file have run.

Next, we mock the Date constructor in our test file.

```Typescript
// imports...

// save the current date for use in the afterAll hook
const OriginalDate = globalThis.Date

// use the proxy API to make the date constructor return a consistent date
beforeAll(() => {
	globalThis.Date = new Proxy(globalThis.Date, {
		construct: () => new OriginalDate('2024-01-01:01:00'),
	})
})

// cleanup - restore the date costructor back to the original
afterAll(() => {
	globalThis.Date = OriginalDate
})
```

[The proxy API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) allows us to
spy on any object. We use it to spy on the global Date object and make it return a consistent date. For our test, we
are choosing January 1st, 2024 which is a Monday, but if we do not set the time explicitly after 00:00, the returned
date will default to 00:00 which is actually the day before until 00:01. Setting it to 01:00 ensures our test day
will always be Monday.

## Section Elaboration

I learned how to properly structure a test with setup, action, and assert sections.

1. setup—anything that needs to be done to put the test code in the correct state for testing
2. action - run the test code and store the results to compare with in the "assert" section
3. assert—compare results with expected results

I learned about how using test blocks gives us a way to write more declarative titles and get more specific test
results, so it is easier to track down problems when a test fails.

I learned about the difference between "in source testing" and the typical testing we see where tests are in test
files, separate from implementation code. "in source" tests might be handy for quickly testing something, but they
cause import issues and can be hard to keep up with as the application grows.

I learned about the beforeAll and afterAll hooks. beforeAll is for handling side effects such as mocking functions,
modules, and network requests that need to be in a certain state for all tests in the test suite. The afterAll hook
is for cleaning up at the end of the test suite by undoing/resetting anything that we did in the "beforeAll" hook.

I learned all of this in much more detail from the linked articles and by building a custom makeshift test runner
throughout the course. There is so much more detail. I took down 350 lines of notes in a markdown file.

# Testing Asynchronous Code

## Awaiting Promises

We have not accounted for async callbacks yet in the test function. This results in a flaky test result that fails
for the wrong reason and causes confusion about the real reason the test will fail even after we fix the async problem.

At first, the response shows that the test passed, and then it is followed by the tests actual error message output.
This is because the test function did not wait for the response from our async callback before moving on.

To fix the first issue, we need to make the test function async and tell it to wait for the callback response.

```Typescript
globalThis.test = async function (title, callback) {
	try {
		await callback()
		console.log(`✓ ${title}`)
	} catch (error) {
		console.error(`✗ ${title}`)
		console.error(error, '\n')
	}
}
```

In modern Javascript, you can await a promise or a non-promise function. The IDE will tell you that "await" has no
effect which you can ignore. But if you like (which I do), you can modify the return type for the test function to
make this warning go away.

```Typescript
var test: (title: string, callback: () => void | Promise<unknown>) => void
```

The type of promise must be unknown because the test function cannot know until runtime.

Now that our test function is properly awaiting async responses, we immediately see the failed test error message,
telling us that the "name" key is undefined. We can focus on determining whether the test or the implementation code
is wrong at this point. Normally, the rule is to trust the test. Actually, the rule is to test the intention. We can
see by the obvious intention in the test that we expect a first name response. This is exactly what the
implementation code is returning. But in our test, we are looking for a "name" key instead of a "firstName" key. Our
test is wrong in this case. Of course, one should always double-check.

## Promise Rejections

Our new greetByResponse function handles the possibility of the user being undefined. We need to write a test to
test that the intent is to reject the promise. In other words, we expect this test to throw an error.

```Typescript
export async function greetByResponse(response: Response) {
	if (typeof response === 'undefined') {
		throw new Error('Failed to greet the user: no user response provided')
	}

	const user = await response.json()
	return greet(user.firstName)
}
```

First, add the type to get some assistance while creating the rejects.toThrow method.

```Typescript
interface Assertions {
	toBe(expected: unknown): void
	rejects: {
		toThrow(expected: Error): Promise<void>
	}
}
```

> [!NOTE]
>
> In earlier versions of the Assertions interface I simply had `[K: string]: (expected: unknown): void` as the type,
> which worked for methods with the same signature but doesn't work for the "rejects" object. It is better to create a
> type for each method anyway because then we can see a list of available methods in suggestions in the IDE.

Now create the "rejects" object and the toThrow method.

```Typescript
rejects: {
    toThrow(expected) {
        if (!(actual instanceof Promise)) {
            throw new Error('Expected to receive a Promise')
        }

        return actual
            .then(() => {
                throw new Error('Expected promise to reject.')
            })
            .catch(error => {
                if (error.message !== expected.message) {
                    throw new Error(
                        `Expected error message to be "${expected.message}" but got "${error.message}"`,
                    )
                }
            })
    },
},
```

What the function does:

1. Checks if actual is a promise and throws an error if it's not.
2. Returns actual.
   We expect a rejection, so if the "then" block is reached, we want to throw an error.
   In the "catch" block, we compare the rejection message to make sure it is throwing the expected message.

Here is the async/await syntax for the toThrow method if you don't like the then/catch syntax.

```Typescript
rejects: {
    async toThrow(expected) {
        if (!(actual instanceof Promise)) {
            throw new Error(`Expected ${actual} to be a promise`)
        }

        try {
            await actual
            throw new Error(`Expected ${actual} to reject but it didn't`)
        } catch (error) {
            if (error.message !== expected.message) {
                throw new Error(
                    `Expected ${error.message} to equal to ${expected.message}`,
                )
            }
        }
    },
},
```

## Awaiting Side Effects

For this example, we have a NotificationsManager class which has a "notifications" array and a showNotification method.
The showNotification method calls the getByResponse method and then pushes the notification to the "notifications"
array.

For testing the intent, we are testing that the "notifications" array contains the notification. The problem here is
that the showNotification method has to wait for the async greetByResponse method to resolve, and then we need to wait
for the notifications array to be pushed to before we can test it. This is similar to waiting for a React/Vue
component to update the DOM because of a state change. We would need to wait for the DOM to be updated before we
could test it.

To allow us to test side effects like this, we create a waitFor method that takes the callback and an optional
maxRetries parameter that defaults to 5. The method retries calling the async callback until it resolves or the
maxRetries is reached whichever one comes first.

The type of the waitFor method is

```Typescript
	var waitFor: (callback: () => void, maxRetries?: number) => Promise<void>
```

And the method is

```Typescript
globalThis.waitFor = async function (callback: () => void, maxRetries = 5) {
	let retries = 0
	while (retries < maxRetries) {
		try {
			retries++
			callback()
			return
		} catch (error) {
			if (retries === maxRetries) {
				throw error
			}
			await new Promise(resolve => setTimeout(resolve, 250))
		}
	}
}
```

In the test, we use it like this.

```Typescript
test('displays a notification when a new user joins', async () => {
	const manager = new NotificationManager()
	manager.showNotification(Response.json({ firstName: 'Kate' }))

	await waitFor(() =>
		expect(manager.notifications[0]).toBe('Hello, Kate! Happy, Monday.'),
	)
})
```

## waitFor vs sleep

It would be easier to replace the "waitFor" method with a sleep function, but there is a problem with this. The sleep
function would be waiting for and relying on time. It might work in this situation but is a recipe for flaky tests.

The waitFor method on the other hand is waiting on state. While it does have a 250ms timer in use for the retries, it
will continue to retry until the max tries are used up. But it will pass or fail on each try, based on the state
matching the expected state which is our intent.

## Section Elaboration

In this section, I learned about how to wait for promises, how to test for expected promise rejections, and how to
wait for side effects. I also learned another important critical thinking concept about how our waitFor method waits
for an expected state, but the easier sleep method would wait for time and how this can lead to flaky tests and false
positives.

### Waiting for Promises:

We convert our test method into an async function that can receive either a promise or a non-promise callback. In
modern Javascript, we can await either one without any errors. Typescript does not seem to have an issue with it either.
The IDE will warn that "await has no effect on the type of this expression" when calling `await callback()`. This
warning can be ignored, but for completeness since this is a Typescript project, I modified the type which lets my
IDE know that "callback" might be a promise.

```Typescript
var test: (title: string, callback: () => void | Promise<void>) => void
```

### Promise Rejections

In this section, we create the toThrow method which will cause our test to fail if the callback is not a promise. It
will also fail if the promise resolves because our expectation is that it will be rejected. Finally, if the promise
rejects, but the error message is not the one we are expecting, the test will fail.

### Awaiting Side Effects

We create a waitFor method to test state that relies on other state changing first. In the example, our
showNotification method calls the async greetByResponse method and then pushes the correct message to an array.
The thing we test is that the correct message is in the array, but this cannot happen until after greetByResponse has
resolved. So in this context, the greetByResponse method is the side effect. The waitFor method will retry a set
number of times until the side effect has resolved, and the showNotification method has pushed the message to the
array. Then the waitFor method will resolve, so our test can show its pass or fail output.

The example side effect can be compared to waiting for a React/Vue component to update the DOM due to a side effect
such as state change or waiting for a fetch call.

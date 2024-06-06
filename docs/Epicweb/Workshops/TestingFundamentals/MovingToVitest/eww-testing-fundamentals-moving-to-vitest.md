# Moving to Vitest

[Vitest](https://vitest.dev/) is a high-performance test runner powered by [Vite](https://vitejs.dev/) and compatible
with [Jest](https://jestjs.io/)

Vitest provides the methods we created in this course and a lot more. The methods are already globally available.
There is also an "it" method that can be used in place of "test."

> [!NOTE]
>
> waitFor can now be replaced with vi.waitFor. There is also a vi.waitUntil.
> These are inspired by testing-library waitFor.
> [vi waitFor](https://vitest.dev/api/vi.html#vi-waitfor)
> [testing-library waitFor](https://testing-library.com/docs/dom-testing-library/api-async/#waitfor)

### Basic Configuration

```Typescript
    // vitest.config.ts
    import { defineConfig } from 'vitest/config'

    export default defineConfig({
        test: {
            globals: true,
        },
    })
```

## Steps to Migrate

1. Delete the setup.ts file. We don't need any of that anymore.
2. Install Vitest. `npm i -D vitest`
3. Create vitest.config.ts with the [basic configuration](#basic-configuration)
4. Add types to the compiler options in your tsconfig.json file `"types": ["vitest/globals"]`
5. Delete the waitFor method in the test file. We'll use the one from vitest
6. Change the code in the test from `await waitFor...` to `await vi.waitFor...`
7. Run vitest `npx vitest`

Vitest will run and all tests will pass. Vitest has "watch" mode enabled by default.

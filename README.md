![@lambda-middleware](assets/lambda-middleware-logo.png)

[![open issues](https://img.shields.io/github/issues-raw/dbartholomae/lambda-middleware.svg)](https://github.com/dbartholomae/lambda-middleware/issues)
[![debug](https://img.shields.io/badge/debug-blue.svg)](https://github.com/visionmedia/debug#readme)
[![build status](https://github.com/dbartholomae/lambda-middleware/workflows/.github/workflows/build.yml/badge.svg?branch=main)](https://github.com/dbartholomae/lambda-middleware/actions?query=workflow%3A.github%2Fworkflows%2Fbuild.yml)
[![codecov](https://codecov.io/gh/dbartholomae/lambda-middleware/branch/main/graph/badge.svg)](https://codecov.io/gh/dbartholomae/lambda-middleware)
[![CLA assistant](https://cla-assistant.io/readme/badge/dbartholomae/lambda-middleware)](https://cla-assistant.io/dbartholomae/lambda-middleware)

This monorepo is a collection of middleware for AWS lambda functions.

## Middlewares

* [@lambda-middleware/class-validator](packages/class-validator): A validation middleware for AWS http lambda functions
  based on class-validator. [![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fclass-validator.svg)](https://npmjs.org/package/@lambda-middleware/class-validator)
* [@lambda-middleware/compose](packages/compose): A compose function for functional lambda middleware. [![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fcompose.svg)](https://npmjs.org/package/@lambda-middleware/compose)
* [@lambda-middleware/http-error-handler](packages/http-error-handler): An error handler middleware for AWS http lambda
  functions.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fhttp-error-handler.svg)](https://npmjs.org/package/@lambda-middleware/http-error-handler)
* [@lambda-middleware/ie-no-open](packages/ie-no-open): A middleware for adding the download options no-open header to
  AWS lambdas.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fie-no-open.svg)](https://npmjs.org/package/@lambda-middleware/ie-no-open)
* [@lambda-middleware/json-serializer](packages/json-serializer): A middleware for AWS http lambda functions to
  serialize JSON responses.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fjson-serializer.svg)](https://npmjs.org/package/@lambda-middleware/json-serializer)
* [@lambda-middleware/json-deserializer](packages/json-deserializer): A middleware for AWS http lambda functions to
  deserialize JSON request bodies.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fjson-deserializer.svg)](https://npmjs.org/package/@lambda-middleware/json-deserializer)
* [@lambda-middleware/jwt-auth](packages/jwt-auth): A middleware for AWS http lambda functions to verify JWT auth
  tokens inspired by express-jwt.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fjwt-auth.svg)](https://npmjs.org/package/@lambda-middleware/jwt-auth)
* [@lambda-middleware/middy-adaptor](packages/middy-adaptor): An adaptor to use middy middleware as functional
  middleware.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fmiddy-adaptor.svg)](https://npmjs.org/package/@lambda-middleware/middy-adaptor)
* [@lambda-middleware/no-sniff](packages/no-sniff): A middleware for adding the content type options no-sniff header
  to AWS lambdas.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fno-sniff.svg)](https://npmjs.org/package/@lambda-middleware/no-sniff)
* [@lambda-middleware/http-header-normalizer](packages/http-header-normalizer): Middleware for AWS lambdas that
  normalizes headers to lower-case.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fhttp-header-normalizer.svg)](https://npmjs.org/package/@lambda-middleware/http-header-normalizer)
* [@lambda-middleware/do-not-wait](packages/do-not-wait): AWS lambda middleware to prevent Lambda from timing out
  because of processes running after returning a value.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fdo-not-wait.svg)](https://npmjs.org/package/@lambda-middleware/do-not-wait)
* [@lambda-middleware/cors](packages/cors): AWS lambda middleware for automatically adding CORS headers.[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Fcors.svg)](https://npmjs.org/package/@lambda-middleware/cors)

## Other packages

Furthermore there is utility collection available at [@lambda-middleware/utils](packages/utils).[![downloads](https://img.shields.io/npm/dw/%40lambda-middleware%2Futils.svg)](https://npmjs.org/package/@lambda-middleware/utils)

## Usage

Each middleware is a higher-order function that can be wrapped around the handler function.

```typescript
export const handler = someMiddleware()(() => {
  return {
    body: '',
    statusCode: 200
  }
})
```

Each middleware is build as
```typescript
(options) => (handler) => (event, context) => Response
```

This means that middleware can be composed and piped like any other function with only one parameter (the handler).
This library contains [a helper for composing](packages/compose), but [any](https://lodash.com/docs/4.17.15#flowRight)
[other](https://ramdajs.com/docs/#compose) [implementation](https://github.com/tc39/proposal-pipeline-operator) should
work as well.

```typescript
export const handler = compose(
  someMiddleware(),
  someOtherMiddleware(),
  aThirdMiddleware()
)(() => {
  return {
    body: '',
    statusCode: 200
  }
})
```

There's a [known issue with TypeScript](https://github.com/microsoft/TypeScript/issues/29904) that pipe and compose functions cannot
infer types correctly if the innermost function is generic (in this case the last argument to `compose`).
If you use TypeScript in strict mode, you can instead use the `composeHandler` function exported from `@lambda-middleware/compose`:

```typescript
export const handler = composeHandler(
  someMiddleware(),
  someOtherMiddleware(),
  aThirdMiddleware(),
  () => {
    return {
      body: '',
      statusCode: 200
    }
  }
)
```

Composing middleware is equivalent to calling it nested:
```typescript
export const handler =
  someMiddleware()(
    someOtherMiddleware()(
      aThirdMiddleware()(() => {
        return {
          body: '',
          statusCode: 200
        }
      })
    )
  )
```

The order of composition can be relevant. When using a helper to do the composition, check, in which order the functions
are applied. Most of the time TypeScript should be able to warn you, if the order is wrong.

Imagine middleware as an onion around your function: The outermost middleware will get called first before the handler
starts, and last after the handler finishes or throws an error. In our example above the order in which middleware gets
executed therefore would be:
```
someMiddleware
  someOtherMiddleware
    aThirdMiddleware
      the handler
    aThirdMiddleware
  someOtherMiddleware
someMiddleware
```
This means that middleware which transforms the input for the handler will be executed top to bottom, while middleware
that transforms the response will be called bottom to top.

## Writing your own middleware

If you want to write your own middleware, check the existing examples and feel free to borrow some of the tests for
inspiration. The general idea for a middleware is the following:
```typescript
const myMiddleware = (optionsForMyMiddleware) => (handler) => async (event, context) => {
  try {
    const modifiedEvent = doSomethingBeforeCallingTheHandler(event)
    const response = await handler(modifiedEvent, context)
    const modifiedResponse = doSomethingAfterCallingTheHandler(response)
    return modifiedResponse
  } catch (error) {
    const modifiedError = doSomethingInCaseOfAnError(error)
    throw modifiedError
  }
}
```
Usually the same middleware should not need to do something before the handler, after the handler and on error.
Creating separated middlewares for these cases keeps them more versatile. But cases that require multiple steps are
supported as well.

Since the middlewares only uses function composition, TypeScript can offer extensive typing support to let you know
how the middleware changed. When adding your own middleware it is recommended to use generics to avoid losing type
information.

Instead of
```typescript
const bodyParser = () =>
  (handler: PromiseHandler<Omit<APIGatewayProxyEvent, body> & { body: object}, APIGatewayProxyResult>): PromiseHandler<APIGatewayProxyEvent, APIGatewayProxyResult> =>
  async (event: E, context: Context) => {
  return handler({ ...event, body: JSON.parse(event.body) }, context)
}
```
use
```typescript
const bodyParser = () =>
  <E extends APIGatewayProxyEvent>(handler: PromiseHandler<Omit<E, body> & { body: object}, APIGatewayProxyResult>): PromiseHandler<E, APIGatewayProxyResult> =>
  async (event: E, context: Context) => {
  return handler({ ...event, body: JSON.parse(event.body) }, context)
}
```
so that if multiple middlewares change the event, the resulting type will have all changes and not just the latest.

## Contributing

If you want to contribute to the project, please read our [contributing guidelines](CONTRIBUTING.md) first.

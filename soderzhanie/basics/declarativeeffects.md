# 2.2 Декларативные эффекты

В `redux-saga`, Саги реализованы функцией-генератором. Что-бы выразить Saga логику, мы производим(yield) обычный JavaScript объект из генератора. Мы называем такие объекты — эффектами ( [_Effects_](https://redux-saga.js.org/docs/api/#effect-creators) ). Эффект — это объект который содержит некую информацию, которая в свою очередь интерпритируется middleware прослойкой. Вы можете представить эффект как
инструкцию для middleware которая выполнит некую операцию (к примеру, вызвать какую-либо асинхронную функцию, задиспатчить экшн в стор, и т.д\)


Что-бы создать эффект, используется функция предоставленная библеотекой внутри `redux-saga/effects` пакета.

В этом и следующих разделах, мы познакомим вас с основными эффектами. И посмотрим как этот концепт позволяет сагам быть легко 
тестируемыми.

Саги могут производить эффекты в различных формах. Легчайший способ — произвести промис.

Например, предположим, что у нас есть сага которая "следит" за `PRODUCTS_REQUESTED` экшином. При каждом совпашем/подходящем экшине, "следящая" сага запускает таску для запроса списка продуктов из сервера.

```javascript
import { takeEvery } from 'redux-saga/effects'
import Api from './path/to/api'

function* watchFetchProducts() {
  yield takeEvery('PRODUCTS_REQUESTED', fetchProducts)
}

function* fetchProducts() {
  const products = yield Api.fetch('/products')
  console.log(products)
}
```

In the example above, we are invoking `Api.fetch` directly from inside the Generator \(In Generator functions, any expression at the right of `yield` is evaluated then the result is yielded to the caller\).

`Api.fetch('/products')` triggers an AJAX request and returns a Promise that will resolve with the resolved response, the AJAX request will be executed immediately. Simple and idiomatic, but...

Suppose we want to test the generator above:

```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??) // what do we expect ?
```

We want to check the result of the first value yielded by the generator. In our case it's the result of running `Api.fetch('/products')` which is a Promise . Executing the real service during tests is neither a viable nor practical approach, so we have to _mock_ the `Api.fetch` function, i.e. we'll have to replace the real function with a fake one which doesn't actually run the AJAX request but only checks that we've called `Api.fetch` with the right arguments \(`'/products'` in our case\).

Mocks make testing more difficult and less reliable. On the other hand, functions that return values are easier to test, since we can use a simple `equal()` to check the result. This is the way to write the most reliable tests.

Not convinced? I encourage you to read [Eric Elliott's article](https://medium.com/javascript-scene/what-every-unit-test-needs-f6cd34d9836d#.4ttnnzpgc):

> \(...\)`equal()`, by nature answers the two most important questions every unit test must answer, but most don’t:
>
> * What is the actual output?
> * What is the expected output?
>
> If you finish a test without answering those two questions, you don’t have a real unit test. You have a sloppy, half-baked test.

What we actually need to do is make sure the `fetchProducts` task yields a call with the right function and the right arguments.

Instead of invoking the asynchronous function directly from inside the Generator, **we can yield only a description of the function invocation**. i.e. We'll yield an object which looks like

```javascript
// Effect -> call the function Api.fetch with `./products` as argument
{
  CALL: {
    fn: Api.fetch,
    args: ['./products']
  }
}
```

Put another way, the Generator will yield plain Objects containing _instructions_, and the `redux-saga` middleware will take care of executing those instructions and giving back the result of their execution to the Generator. This way, when testing the Generator, all we need to do is to check that it yields the expected instruction by doing a simple `deepEqual` on the yielded Object.

For this reason, the library provides a different way to perform asynchronous calls.

```javascript
import { call } from 'redux-saga/effects'

function* fetchProducts() {
  const products = yield call(Api.fetch, '/products')
  // ...
}
```

We're using now the `call(fn, ...args)` function. **The difference from the preceding example is that now we're not executing the fetch call immediately, instead,** `call` **creates a description of the effect**. Just as in Redux you use action creators to create a plain object describing the action that will get executed by the Store, `call` creates a plain object describing the function call. The redux-saga middleware takes care of executing the function call and resuming the generator with the resolved response.

This allows us to easily test the Generator outside the Redux environment. Because `call` is just a function which returns a plain Object.

```javascript
import { call } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)
```

Now we don't need to mock anything, and a basic equality test will suffice.

The advantage of those _declarative calls_ is that we can test all the logic inside a Saga by iterating over the Generator and doing a `deepEqual` test on the values yielded successively. This is a real benefit, as your complex asynchronous operations are no longer black boxes, and you can test in detail their operational logic no matter how complex it is.

`call` also supports invoking object methods, you can provide a `this` context to the invoked functions using the following form:

```javascript
yield call([obj, obj.method], arg1, arg2, ...) // as if we did obj.method(arg1, arg2 ...)
```

`apply` is an alias for the method invocation form

```javascript
yield apply(obj, obj.method, [arg1, arg2, ...])
```

`call` and `apply` are well suited for functions that return Promise results. Another function `cps` can be used to handle Node style functions \(e.g. `fn(...args, callback)` where `callback` is of the form `(error, result) => ()`\). `cps` stands for Continuation Passing Style.

For example:

```javascript
import { cps } from 'redux-saga/effects'

const content = yield cps(readFile, '/path/to/file')
```

And of course you can test it just like you test `call`:

```javascript
import { cps } from 'redux-saga/effects'

const iterator = fetchSaga()
assert.deepEqual(iterator.next().value, cps(readFile, '/path/to/file') )
```

`cps` also supports the same method invocation form as `call`.

A full list of declarative effects can be found in the [API reference](https://redux-saga.js.org/docs/api/#effect-creators).


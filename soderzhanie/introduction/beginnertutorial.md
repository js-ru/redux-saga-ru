# 1.1 Руководство для начинающих

## Цели этого урока

Это руководство попытается представить redux-saga в (надеюсь) доступной форме.

Для нашего начального урока мы будем использовать обычный демо-счётчик из Redux репозитория. Приложение довольно простое, но оно хорошо подходит для иллюстрации основных концепций redux-saga, не теряясь при этом в излишних деталях.

### Начальная настройка

Перед тем, как мы начнём, склонируйте [учебный репозиторий](https://github.com/redux-saga/redux-saga-beginner-tutorial).

> Окончательный код этого урока находится в ветке `sagas`.

Затем в командной строке введите:

```bash
$ cd redux-saga-beginner-tutorial
$ npm install
```

Для запуска приложения введите:

```bash
$ npm start
```

Мы начинаем с самого простого варианта использования: 2 кнопки для `Увеличения` и `Уменьшения` счётчика. Позже мы познакомимся с асинхронными вызовами.

Если запуск прошёл успешно, вы увидите 2 кнопки `Increment` и `Decrement` вместе с сообщением ниже `Counter: 0`.

> В случае, если вы столкнулись с проблемами при запуске приложения, не стесняйтесь создавать issue в [репозитории учебника](https://github.com/redux-saga/redux-saga-beginner-tutorial/issues).

## Привет, Саги!

Мы собираемся создать нашу первую Сагу. По традиции мы напишем нашу версию 'Привет, мир' для Саг.

Создайте файл `sagas.js` и добавьте в него следующий кусок кода:

```javascript
export default function* helloSaga() {
  console.log('Hello Sagas!')
}
```

Он совсем не страшный, это просто обычная функция \(кроме `*`\). Всё, что она делает - это выводит приветственное сообщение в консоль.

Для запуска нашей Саги нужно:

* создать промежуточный слой (Saga middleware) со списком наших Саг, которые мы хотим запустить (пока что у нас есть только `helloSaga`)
* подключить этот промежуточный слой к хранилищу Redux (Redux store)

 Мы внесём следующие изменения в файл `main.js`:

```javascript
// ...
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

// ...
import { helloSaga } from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(helloSaga)

const action = type => store.dispatch({type})

// остальное не изменилось
```

Сначала мы импортируем нашу Сагу из модуля `./sagas`. Затем мы создаём промежуточный слой (middleware), используя фабричную функцию `createSagaMiddleware`, которая экспортируется из библиотеки `redux-saga`.

Перед запуском нашей `helloSaga` мы должны подключить наш промежуточный слой (middleware) к Хранилищу (Store), используя `applyMiddleware`. После этого мы сможем использовать `sagaMiddleware.run(helloSaga)` для запуска нашей Саги.

Пока что наша Сага не делает ничего особенного. Она просто выводит сообщение, затем завершается.

## Создание асинхронных вызовов

Теперь давайте добавим что-нибудь, чтобы приблизиться к оригинальному демо-счётчику. Чтобы проиллюстрировать асинхронные вызовы, мы добавим ещё одну кнопку, которая будет увеличивать счётчик через 1 секунду после клика.

Перво-наперво мы добавим кнопку и функцию обратного вызова `onIncrementAsync` в UI компонент.

```javascript
const Counter = ({ value, onIncrement, onDecrement, onIncrementAsync }) =>
  <div>
    <button onClick={onIncrementAsync}>
      Increment after 1 second
    </button>
    {' '}
    <button onClick={onIncrement}>
      Increment
    </button>
    {' '}
    <button onClick={onDecrement}>
      Decrement
    </button>
    <hr />
    <div>
      Clicked: {value} times
    </div>
  </div>
```

Затем мы должны подключить `onIncrementAsync` Компонента к действию Хранилища (Store action)

Мы изменим модуль `main.js` следующим образом:

```javascript
function render() {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => action('INCREMENT')}
      onDecrement={() => action('DECREMENT')}
      onIncrementAsync={() => action('INCREMENT_ASYNC')} />,
    document.getElementById('root')
  )
}
```

Обратите внимание, что в отличие от redux-thunk, наш компонент отправляет (dispatches) действие (action) в виде простого объекта.

Теперь мы представим другую Сагу для выполнения асинхронного вызова. Наш вариант использования выглядит следующим образом:

> На каждое `INCREMENT_ASYNC` действие (action), мы хотим запустить задачу, которая будет делать следующее:
>
> * Подождать 1 секунду, затем увеличить счётчик

Добавьте следующий код в модуль `sagas.js`:

```javascript
import { put, takeEvery } from 'redux-saga/effects'

const delay = (ms) => new Promise(res => setTimeout(res, ms))

// ...

// Наша Сага-рабочий (worker Saga): будет выполнять асинхронную задачу увеличения счётчика
export function* incrementAsync() {
  yield delay(1000)
  yield put({ type: 'INCREMENT' })
}

// Наша Сага-наблюдатель: создаёт новые incrementAsync задачи на каждом INCREMENT_ASYNC
export function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}
```

Время для объяснений.

Мы создали функцию `delay`, которая возвращает [обещание (Promise)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), которое в свою очередь разрешается (resolve) через указанное количество миллисекунд. Мы будем использовать эту функцию, чтобы _заблокировать_ Генератор.

Саги реализованы на [функциях-генераторах](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*), которые _передают (yield)_ объекты промежуточному слою саг (redux-saga middleware). Переданные объекты это своего рода инструкции, которые должны интерпретироваться промежуточным слоем (middleware). Когда Обещание (Promise) передаётся промежуточному слою, тот в свою очередь приостанавливает Сагу до тех пор, пока Обещание не выполнится. В примере выше Сага `incrementAsync` приостанавливается до тех пор, пока Обещание, возвращённое функцией `delay` не разрешится (resolve), что произойдёт через 1 секунду.

Как только Обещание разрешится, промежуточный слой возобновит выполнение кода Саги до следующего yield. В этом примере следующее заявление - это другой переданный (yielded) объект, результат вызова `put({type: 'INCREMENT'})`, который сообщает промежуточному слою отправить (dispatch) действие `INCREMENT`.

`put` это пример того, что мы называем _Эффектом_. Эффекты - это простые JavaScript объекты. Они содержат инструкции, которые должны быть выполнены промежуточным слоем. Когда промежуточный слой извлекает Эффект, переданный Сагой, Сага приостанавливается до тех пор, пока Эффект не выполнится.

Подведём итог. Сага `incrementAsync` спит 1 секунду при помощи вызова `delay(1000)`, затем отправляет (dispatches) действие `INCREMENT`.

Теперь мы создадим другую Сагу `watchIncrementAsync`. Мы будет использовать `takeEvery` - это вспомогательная функция, которая предоставляется `redux-saga` для прослушивания отправленных `INCREMENT_ASYNC` действий и запуска `incrementAsync` при каждой отправке.

Теперь у нас 2 Саги, и нам нужно запустить их одновременно. Чтобы сделать это, мы добавим `rootSaga`, которая отвечает за запуск других наших Саг. Отредактируйте файл `sagas.js` следующим образом:

```javascript
import { put, takeEvery, all } from 'redux-saga/effects'

const delay = (ms) => new Promise(res => setTimeout(res, ms))

function* helloSaga() {
  console.log('Hello Sagas!')
}

export function* incrementAsync() {
  yield delay(1000)
  yield put({ type: 'INCREMENT' })
}

export function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}

// обратите внимание, как мы экспортируем rootSaga
// единая точка входа для запуска всех Саг одновременно
export default function* rootSaga() {
  yield all([
    helloSaga(),
    watchIncrementAsync()
  ])
}
```

Эта Сага передаёт массив с результатом вызова двух наших саг: `helloSaga` и `watchIncrementAsync`. Другими словами, два полученных Генератора будут запущены параллельно. Теперь нам осталось только в файле `main.js` вызвать `sagaMiddleware.run` и передать в качестве аргумента `rootSaga`.

```javascript
// ...
import rootSaga from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = ...
sagaMiddleware.run(rootSaga)

// ...
```

## Делаем наш код тестируемым

Мы хотим протестировать нашу `incrementAsync` Сагу чтобы убедиться, что она выполняет желаемое задание.

Создадим другой файл `sagas.spec.js`:

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  // Что теперь?
});
```

`incrementAsync` - это функция-генератор. Во время запуска она возвращает объект-итератор, а метод `next` итератора в свою очередь возвращает объект следующей формы:

```javascript
gen.next() // => { done: boolean, value: any }
```

Поле `value` содержит полученное (yielded) выражение, т.е. результат выражения после `yield`. Поле `done` показывает, завершился ли генератор или есть ещё выражения 'yield'.

В случае с `incrementAsync` генератор передаёт (yields) 2 значения последовательно:

1. `yield delay(1000)`
2. `yield put({type: 'INCREMENT'})`

Поэтому если мы вызовем метод next генератора 3 раза подряд, то мы получим следующий результат:

```javascript
gen.next() // => { done: false, value: <result of calling delay(1000)> }
gen.next() // => { done: false, value: <result of calling put({type: 'INCREMENT'})> }
gen.next() // => { done: true, value: undefined }
```

Первые 2 вызова возвращают результат yield выражений. На 3-й вызов, поскольку yield больше нет, в поле `done` устанавливается значение true. И поскольку Генератор `incrementAsync` не возвращает ничего \(нет директивы `return`\), поле `value` устанавливается в `undefined`.

Итак, чтобы проверить логику внутри `incrementAsync`, нам нужно будет перебрать возвращённый Генератор и проверить значения, переданные (yielded) генератором.

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  assert.deepEqual(
    gen.next(),
    { done: false, value: ??? },
    'incrementAsync должен вернуть Обещание которое разрешится через 1 секунду'
  )
});
```

Вопрос в том, как мы протестируем возвращённое `delay` значение? Мы не можем сделать простой тест на равенство в Обещаниях (Promises). Если бы `delay` вернула _нормальное_ значение, было бы гораздо проще.

Что ж, `redux-saga` предоставляет способ сделать приведённое выше утверждение возможным. Вместо вызова `delay(1000)` непосредственно внутри `incrementAsync`, мы будем вызывать его _косвенно_ и экспортировать, чтобы сделать возможным последующее глубокое сравнение:

```javascript
import { put, takeEvery, all, call } from 'redux-saga/effects'

export const delay = (ms) => new Promise(res => setTimeout(res, ms))

// ...

export function* incrementAsync() {
  // use the call Effect
  yield call(delay, 1000)
  yield put({ type: 'INCREMENT' })
}
```

Вместо `yield delay(1000)` мы напишем `yield call(delay, 1000)`. В чём разница?

В первом случае yield выражение `delay(1000)` выполняется до того, как оно передаётся в качестве результата `next` \(метод next может вызывать как промежуточный слой (middleware), так и наш тестовый код, который вызывает функцию-генератор и перебирает возвращаемый Генератор\). Так что метод next вернёт Обещание (Promise), как в коде для тестов выше.

Во втором случае yield выражение `call(delay, 1000)` - это то, что возвращается из `next`. `call` как `put`, возвращает Эффект, который указывает промежуточному слою (middleware) вызвать заданную функцию с заданными аргументами. По факту ни `put` ни `call` не выполняют никаких отправок (dispatch) или асинхронных вызовов сами, они возвращают простые JavaScript объекты.

```javascript
put({type: 'INCREMENT'}) // => { PUT: {type: 'INCREMENT'} }
call(delay, 1000)        // => { CALL: {fn: delay, args: [1000]}}
```

Происходит то, что промежуточный слой (middleware) осматривает тип каждого переданного Эффекта, затем решает как выполнить этот Эффект. Если тип Эффекта `PUT`, тогда он отправит (dispatch) действие в Хранилище (Store). Если `CALL` - он вызовет заданную функцию.

Такое разделение между созданием Эффектов и их выполнением позволяет удивительно легко тестировать наш Генератор:

```javascript
import test from 'tape';

import { put, call } from 'redux-saga/effects'
import { incrementAsync, delay } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  assert.deepEqual(
    gen.next().value,
    call(delay, 1000),
    'incrementAsync Saga must call delay(1000)'
  )

  assert.deepEqual(
    gen.next().value,
    put({type: 'INCREMENT'}),
    'incrementAsync Saga must dispatch an INCREMENT action'
  )

  assert.deepEqual(
    gen.next(),
    { done: true, value: undefined },
    'incrementAsync Saga must be done'
  )

  assert.end()
});
```

Так как `put` и `call` возвращают простые объекты, мы можем повторно использовать одни и те же функции в нашем тестовом коде. И чтобы протестировать логику `incrementAsync`, мы перебираем генератор и выполняем `deepEqual` тесты на значениях.

Для запуска вышеуказанного теста выполните:

```bash
$ npm test
```

который должен вернуть результаты в консоли.


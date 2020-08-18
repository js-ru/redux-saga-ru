# 7. Глоссарий

Это глоссарий основных терминов Redux Saga.

## Эффект (Effect)

Это простой JavaScript объект содержащий некоторые инструкции, которые должны выполняться Сага мидлварами.

Вы создаете Эффект, используя фабричные функции, представляемые библиотекой `redux-saga`. Например, вы используете `call(myfunc, 'arg1', 'arg2')` чтобы указать мидлвару вызвать `myfunc('arg1', 'arg2')` и вернуть обратно в Генератор, который вызвал Эффект.

## Задача (Task)

Задача похожа на процесс, работающий в фоновом режиме. В приложении на основе redux-saga может быть несколько параллельно выполняемых задач. Вы создаете задачи используя функцию `fork`

```javascript
import {fork} from "redux-saga/effects"

function* saga() {
  ...
  const task = yield fork(otherSaga, ...args)
  ...
}
```

## Блокирующие/Неблокирующие вызовы (Blocking/Non-blocking call)

Блокирующий вызов значит что Saga вызвала Эффект и будет ожидать результата своего выполнения, прежде чем возобновить выполнение следующей инструкции внутри генератора, выдающего результат.

Неблокирующий вызов означает, что Сага возобновится сразу же после получения Эффекта.
Например

```javascript
import {call, cancel, join, take, put} from "redux-saga/effects"

function* saga() {
  yield take(ACTION)              // Blocking: will wait for the action
  yield call(ApiFn, ...args)      // Blocking: will wait for ApiFn (If ApiFn returns a Promise)
  yield call(otherSaga, ...args)  // Blocking: will wait for otherSaga to terminate

  yield put(...)                   // Non-Blocking: will dispatch within internal scheduler

  const task = yield fork(otherSaga, ...args)  // Non-blocking: will not wait for otherSaga
  yield cancel(task)                           // Non-blocking: will resume immediately
  // or
  yield join(task)                              // Blocking: will wait for the task to terminate
}
```

## Наблюдатель/Воркер (Watcher/Worker)

относится к способу организации потока управления с использованием двух отдельных Саг.

* Наблюдатель: будет следить за отправленными действиями и форкнет воркер для каждого действия
* Воркер: обработает действие и завершит

например

```javascript
function* watcher() {
  while (true) {
    const action = yield take(ACTION)
    yield fork(worker, action.payload)
  }
}

function* worker(payload) {
  // ... do some stuff
}
```


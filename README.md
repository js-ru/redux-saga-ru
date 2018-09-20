<img src='https://redux-saga.js.org/logo/0800/Redux-Saga-Logo-Landscape.png' alt='Redux Logo Landscape' width='800px'>

# redux-saga

[![npm version](https://img.shields.io/npm/v/redux-saga.svg)](https://www.npmjs.com/package/redux-saga)
[![CDNJS](https://img.shields.io/cdnjs/v/redux-saga.svg)](https://cdnjs.com/libraries/redux-saga)
[![npm](https://img.shields.io/npm/dm/redux-saga.svg)](https://www.npmjs.com/package/redux-saga)
[![Build Status](https://travis-ci.org/redux-saga/redux-saga.svg?branch=master)](https://travis-ci.org/redux-saga/redux-saga)
[![Join the chat at https://gitter.im/yelouafi/redux-saga](https://badges.gitter.im/yelouafi/redux-saga.svg)](https://gitter.im/yelouafi/redux-saga?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![OpenCollective](https://opencollective.com/redux-saga/backers/badge.svg)](#backers)
[![OpenCollective](https://opencollective.com/redux-saga/sponsors/badge.svg)](#sponsors)

`redux-saga` — это библиотека, которая призвана упростить и улучшить побочные эффектов (т.е. таких действий как асинхронные операции, например, загрузки данных и "грязных" действий, таких как доступ к браузерному кешу), сделать лёгкими в тестировании и лучше справляться с ошибками.

Можно представить это так, что saga — это как отдельный поток в вашем приложении, который отвечает за побочные эффекты. `redux-saga` — это мидлвар redux, что означает, что этот поток может запускаться, останавливаться и отменяться из основного приложения с помощью обычных действий redux, оно имеет доступ к полному состоянию redux приложения и также может диспатчить действия redux.

Библиотека использует концепцию ES6, под названием генераторы, для того, чтобы сделать эти асинхронные потоки легкими для чтения, написания и тестирования. *(если вы не знакомы с этим, [здесь есть некоторые ссылки для ознакомления](https://redux-saga.js.org/docs/ExternalResources.html))* Тем самым, эти асинхронные потоки выглядят, как ваш стандартный синхронный JavaScript-код. (наподобие `async`/`await`, но генераторы имеют несколько отличных возможностей, необходимых нам)

Возможно, вы уже использовали `redux-thunk`, перед тем как обрабатывать ваши выборки данных. В отличие от redux thunk, вы не оказываетесь в аду колбэков, вы можете легко тестировать ваши асинхронные потоки и ваши действия остаются чистыми.

# Начало работы

```sh
$ npm install --save redux-saga
```
или

```sh
$ yarn add redux-saga
```

Альтернативно, вы можете использовать предоставленные UMD сборки напрямую в `<script>` на HTML странице. Смотрите [эту секцию](#using-umd-build-in-the-browser).

## Пример использования

Предположим, что у нас есть интерфейс для извлечения некоторых пользовательских данных с удалённого сервера при нажатии кнопки. (Для краткости, мы просто покажем код запуска экшена.)

```javascript
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
```

Компонент диспатчит действие в виде простого объекта в хранилище. Мы создадим saga, которая слушает все действия `USER_FETCH_REQUESTED` и генерирует вызовы API для извлечения пользовательских данных.

#### `sagas.js`

```javascript
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

// воркер Saga: будет запускаться на действия типа `USER_FETCH_REQUESTED`
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  Запускаем `fetchUser` на каждый задиспатченое действие `USER_FETCH_REQUESTED`.
  Позволяет одновременно получать данные пользователей.
*/
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  В качестве альтернативы вы можете использовать `takeLatest`.

  Не допускает одновременное получение данных пользователей. Если `USER_FETCH_REQUESTED`
  диспатчится в то время когда предыдущий запрос все еще находится в ожидании ответа,
  то этот ожидающий ответа запрос отменяется и срабатывает только последний.
*/
function* mySaga() {
  yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

Для запуска Saga, мы подключим ее к Redux Store, используя мидлвар `redux-saga`.

#### `main.js`

```javascript
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// создаем мидлвар saga
const sagaMiddleware = createSagaMiddleware()
// монтируем его в хранилище
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// затем запускаем saga
sagaMiddleware.run(mySaga)

// отрисовываем приложение
```

# Документация

- [Введение](https://redux-saga.js.org/docs/introduction/BeginnerTutorial.html)
- [Базовые концепции](https://redux-saga.js.org/docs/basics/index.html)
- [Продвинутое использование](https://redux-saga.js.org/docs/advanced/index.html)
- [Рецепты](https://redux-saga.js.org/docs/recipes/index.html)
- [Сторонние ресурсы](https://redux-saga.js.org/docs/ExternalResources.html)
- [Устранение проблем](https://redux-saga.js.org/docs/Troubleshooting.html)
- [Глоссарий](https://redux-saga.js.org/docs/Glossary.html)
- [Справочник по API](https://redux-saga.js.org/docs/api/index.html)

# Переводы

- [Chinese](https://github.com/superRaytin/redux-saga-in-chinese)
- [Traditional Chinese](https://github.com/neighborhood999/redux-saga)
- [Japanese](https://github.com/redux-saga/redux-saga/blob/master/README_ja.md)
- [Korean](https://github.com/redux-saga/redux-saga/blob/master/README_ko.md)
- [Portuguese](https://github.com/joelbarbosa/redux-saga-pt_BR)
- [Russian](https://github.com/redux-saga/redux-saga/blob/master/README_ru.md)

# Использование UMD-сборки в браузере

Также существует сборка **umd** `redux-saga`, доступная в каталоге `dist/`. При использовании umd-сборки, `redux-saga` доступна как `ReduxSaga` в объекте `window`.

версия umd полезна, если вы не используете Webpack или Browserify. Вы можете получить доступ к ней, непосредственно из [unpkg](https://unpkg.com/).

Доступны следующие сборки:

- [https://unpkg.com/redux-saga/dist/redux-saga.js](https://unpkg.com/redux-saga/dist/redux-saga.js)
- [https://unpkg.com/redux-saga/dist/redux-saga.min.js](https://unpkg.com/redux-saga/dist/redux-saga.min.js)

**Важно!** Если ваш целевой браузер не поддерживает *генераторы ES2015*, вам нужно транспилировать код (например, через [babel plugin](https://github.com/facebook/regenerator/tree/master/packages/regenerator-transform)) и предоставить корректный runtime, такой, например, как [этот](https://unpkg.com/regenerator-runtime/runtime.js). Runtime нужно импортировать до **redux-saga**:

```javascript
import 'regenerator-runtime/runtime'
// затем
import sagaMiddleware from 'redux-saga'
```

# Сборка примеров из исходных файлов

```sh
$ git clone https://github.com/redux-saga/redux-saga.git
$ cd redux-saga
$ npm install
$ npm test
```

Ниже приведены примеры, портированные (пока) из репозиториев Redux.

### Примеры счётчика

Есть три примера счётчика.

#### counter-vanilla

Демо, использующее обычный JavaScript и UMD-сборки. Все исходники находятся в `index.html`.

Для запуска примера, просто откройте `index.html` в браузере.

> Важно: ваш браузер должен поддерживать генераторы. Подойдут последние версии Chrome/Firefox/Edge.

#### counter

Демо, использующее `webpack` и высокоуровневое API `takeEvery`.

```sh
$ npm run counter

# тестовый образец для генератора
$ npm run test-counter
```

#### cancellable-counter

Демо, использующее низкоуровневое API для демонстрации отмены задачи.

```sh
$ npm run cancellable-counter
```

### пример с корзиной

```sh
$ npm run shop

# тестовый образец для генератора
$ npm run test-shop
```

### пример с async

```sh
$ npm run async

# тестовый образец для генераторов
$ npm run test-async
```

### реальный пример (с горячей перезагрузкой webpack)

```sh
$ npm run real-world

# извините, тестов пока нет
```

### TypeScript

Redux-Saga с TypeScript требует `DOM.Iterable` или `ES2015.Iterable`. Если ваш `target` это `ES6`, у вас, скорее всего, уже всё настроено, однако для `ES5` вам потребуется самостоятельно добавить его.
Посмотрите файл `tsconfig.json` и официальную документацию с <a href="https://www.typescriptlang.org/docs/handbook/compiler-options.html">опциями компилятора</a>.

### Логотип

Вы можете найти официальный логотип Redux-Saga с различными вариантами в [каталоге logo](https://github.com/redux-saga/redux-saga/tree/master/logo).


### Меценат
Поддержите нас ежемесячными пожертвованиями и помогите нам продолжать нашу деятельность. \[[Стать меценатом](https://opencollective.com/redux-saga#backer)\]

<a href="https://opencollective.com/redux-saga/backer/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/29/avatar.svg"></a>


### Спонсоры
Стань спонсором и получи свой логотип в нашем README на GitHub с ссылкой на ваш сайт. \[[Стать спонсором](https://opencollective.com/redux-saga#sponsor)\]

<a href="https://opencollective.com/redux-saga/sponsor/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/29/avatar.svg"></a>

# 2.1. Использование хелперов Saga

`redux-saga` обеспечивает некоторые вспомогательные эффекты для вызова задач, когда некоторые конкретные действия отправляются в `redux-store`.

Вспомогательные функции построены поверх API нижнего уровня. В расширенном разделе мы увидим, как эти функции могут быть реализованы.

Первая функция `takeEvery` является наиболее знакомой и обеспечивает поведение, аналогичное `redux-thunk`.

Давайте посмотрим на общем примере AJAX. При каждом нажатии на кнопку мы отправляем действие `FETCH_REQUESTED` action. Мы хотим обработать это действие, запустив задачу, которая будет получать некоторые данные с сервера.

Сначала мы создадим задачу, которая будет выполнять асинхронное действие:

```javascript
import { call, put } from 'redux-saga/effects'

export function* fetchData(action) {
   try {
      const data = yield call(Api.fetchUser, action.payload.url)
      yield put({type: "FETCH_SUCCEEDED", data})
   } catch (error) {
      yield put({type: "FETCH_FAILED", error})
   }
}
```

Чтобы запустить вышеуказанную задачу для каждого `FETCH_REQUESTED` действия:

```javascript
import { takeEvery } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeEvery('FETCH_REQUESTED', fetchData)
}
```

В приведенном выше примере takeEvery позволяет одновременно запускать несколько fetchData. В данный момент мы можем запустить новую задачу fetchData, пока есть еще одна или несколько предыдущих задач fetchData, которые еще не завершены.

если мы хотим получить только ответ на последний запрос (например, чтобы всегда отображать последнюю версию данных), мы можем использовать помощник `takeLatest`:

```javascript
import { takeLatest } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeLatest('FETCH_REQUESTED', fetchData)
}
```

В отличие от takeEvery, takeLatest позволяет запускать только одну задачу fetchData в любой момент. И это будет последняя запущенная задача Если предыдущая задача все еще выполняется при запуске другой задачи `fetchData`, предыдущая задача будет автоматически отменена.

Если у вас есть несколько `Sags`, наблюдающих за различными действиями, вы можете создать несколько наблюдателей с помощью этих встроенных функций, которые будут вести себя так, если бы они использовали `fork` \ (мы поговорим о` fork` позже. Сейчас рассмотрим пример, который позволяет нам запускать несколько `Sags` на заднем плане \).

For example:

```javascript
import { takeEvery } from 'redux-saga/effects'

// FETCH_USERS
function* fetchUsers(action) { ... }

// CREATE_USER
function* createUser(action) { ... }

// use them in parallel
export default function* rootSaga() {
  yield takeEvery('FETCH_USERS', fetchUsers)
  yield takeEvery('CREATE_USER', createUser)
}
```


# Nội dụng

- [Epic](#epics)
    - [Epic Example](#epic-example)
        - [Sync](#sync)
        - [Async](#async)

# Epic

> Epic giống Saga trong redux-saga

Epic là một function nhận vào _a stream of actions_ và trả về  _a stream of actions_

```js
    function (action$: Observable<Action>, store: Store): Observable<Action>;
```

Action sẽ đi qua reducer trước rồi sẽ tới Epic (tương tự Saga)

Action được emit sẽ ngay lập tức được dispatch ```store.dispatch()```. Chính vì vậy, _under the hood_,
epic sẽ được dùng như sau:

```js
    // observable.subscribe(x => console.log('Observer got a next value: ' + x));
    epic(action$, store).subscribe(store.dispatch)
```

## Ví dụ về Epic

> Nếu muốn dùng operator thì phải import [Learn more](https://redux-observable.js.org/docs/Troubleshooting.html#rxjs-operators-are-missing-eg-typeerror-actionoftypeswitchmap-is-not-a-function)

### Sync

Epic sau lắng nghe action ```PING``` và dispatch action ```PONG```

```js
    const pingEpic = action$ =>
    action$.filter(action => action.type === 'PING')
        .mapTo({ type: 'PONG' });

    // later...
    dispatch({ type: 'PING' });
```

> Dollar sign ở cuối tên biến để biết biến đó là một stream (RxJS convention)

### Async

Epic sau lắng nghe action ```PING``` và dispatch action ```PONG``` sau 1s

```js
    const pingEpic = action$ =>
    action$.filter(action => action.type === 'PING')
        .delay(1000) // Asynchronously wait 1000ms then continue
        .mapTo({ type: 'PONG' });

    // later...
    dispatch({ type: 'PING' });
```

> Thay vì ```action$.filter(action => action.type === 'PING'```, nên dùng ```action$.ofType('PING')```

### Ví dụ thực tế

```js
import { ajax } from 'rxjs/observable/dom/ajax';

// action creators
const fetchUser = username => ({ type: FETCH_USER, payload: username });
const fetchUserFulfilled = payload => ({ type: FETCH_USER_FULFILLED, payload });

// epic
const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
      ajax.getJSON(`https://api.github.com/users/${action.payload}`)
        .map(response => fetchUserFulfilled(response))
    );

// later...
dispatch(fetchUser('torvalds'));
```

## Sử dụng ```store``` trong Epic

Epic nhận vào tham số thứ 2 là store - một *light version* của Redux store.
Nó chỉ có 2 method ```store.getState()``` và ```store.dispatch()```.

Khi Epic nhận được action, nghĩa là action đó đã đi qua reducer và state đã được update.

Không nên dùng ```store.dispatch()``` trong Epic (bad practice).

## Combine Epic

Ta có thể combine nhiều epic thành 1 rootEpic (giống như yield all để tạo rootSaga)

```js
import { combineEpics } from 'redux-observable';

const rootEpic = combineEpics(
  pingEpic,
  fetchUserEpic
);
```

Note: sử dụng ```combineEpics()``` tương tự với viết như sau:

```js
import { merge } from 'rxjs/observable/merge';

const rootEpic = (action$, store) => merge(
  pingEpic(action$, store),
  fetchUserEpic(action$, store)
);
```

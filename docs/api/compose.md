# `compose(...functions)`

Объединяет функции слева направо.

Это утилита функционального программирования, которая включена в Redux для удобства.   
Вы можете использовать ее, чтобы применить несколько [расширителей хранилища](../Glossary.md#store-enhancer) последовательно.

#### Параметры

  1. (*аргументы*): функции для композиции. Ожидается, что каждая функция принимает один параметр. Ее возвращаемое значение будет представлено в качестве аргумента для функции стоящей слева, и так далее.

#### Возвращает

(*Функция*): конечная функция полученая путем композиции переданных функций слева направо.

#### Пример

В этом примере показано, как использовать `compose` для расширения [хранилища](Store.md) с [`applyMiddleware`](applyMiddleware.md) и несколько инструментов для разработки из пакета [redux devtools](https://github.com/gaearon/redux-devtools).

```js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import * as reducers from '../reducers/index';

let reducer = combineReducers(reducers);
let middleware = [thunk];

let finalCreateStore;

// В продакшене мы хотим использовать только этот посредник.
// В разработке мы хотим использовать некоторые расширители хранилища из redux devtools.
// UglifyJS удалит "мертвый код" в зависимости от среды сборки.

if (process.env.NODE_ENV === 'production') {
  finalCreateStore = applyMiddleware(...middleware)(createStore);
} else {
  finalCreateStore = compose(
    applyMiddleware(...middleware),
    require('redux-devtools').devTools(),
    require('redux-devtools').persistState(
      window.location.href.match(/[?&]debug_session=([^&]+)\b/)
    )
  )(createStore);

  // Некоторы код без хелперов `compose`:
  //
  // finalCreateStore = applyMiddleware(middleware)(
  //   require('redux-devtools').devTools()(
  //     require('redux-devtools').persistState(window.location.href.match(/[?&]debug_session=([^&]+)\b/))()
  //   )
  // )(createStore);
}

let store = finalCreateStore(reducer);
```

#### Советы

  * `compose` позволяет вам писать глубоко вложенные функция преобразований без дрейфа вправо (rightward drift) в коде. Не предоставляйте ей слишком многого!
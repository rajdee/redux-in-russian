# `compose(...functions)`

Объединяет функции слева направо.

Это утилита функционального программирования, которая включена в Redux для удобства.   
Вы можете использовать ее, чтобы применить несколько [расширителей хранилища](../Glossary.md#store-enhancer) последовательно.

#### Параметры

  1. (*аргументы*): функции для композиции. Ожидается, что каждая функция принимает один параметр. Ее возвращаемое значение будет представлено в качестве аргумента для функции стоящей слева, и так далее.

#### Возвращает

(*Функция*): конечная функция полученая путем композиции переданных функций справа налево.

#### Пример

В этом примере показано, как использовать `compose` для расширения [хранилища](Store.md) с [`applyMiddleware`](applyMiddleware.md) и несколько инструментов для разработки из пакета [redux devtools](https://github.com/gaearon/redux-devtools).

```js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import DevTools from './containers/DevTools'
import reducer from '../reducers/index'

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunk),
    DevTools.instrument()
  )
)
```

#### Советы

  * `compose` позволяет вам писать глубоко вложенные функции преобразований без дрейфа вправо (rightward drift) в коде. Не предоставляйте ей слишком многого!

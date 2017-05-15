# Использование оператора расширения

Поскольку одним из основных постулатов Redux является "никогда не изменять состояние напрямую", вам часто придется использовать [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) для создания копий объектов с новыми или измененными значениями. Например, в примере `todoApp` ниже используется `Object.assign()` для возвращения нового `состояния` объекта с обновленным `visibilityFilter` полем:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

Несмотря на эффективность, `Object.assign()` может довольно быстро сделать простые редюсеры трудно читаемыми.

Альтернативный вариант — использовать [object spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread), предложенный для следующих версий JavaScript, который позволяет вам использовать spread-оператор (`...`) для копирования перечисляемых свойств одного объекта в другой в более сжатой форме записи. Оператор расширения концептуально похож на [array spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) ES6. Мы можем упростить пример `todoApp`, используя object spread syntax:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return { ...state, visibilityFilter: action.filter }
    default:
      return state
  }
}
```

Преимущество использования оператора расширения становится более очевидно при составлении сложных объектов. В примере ниже `getAddedIds` сопоставляет массив с `ids` массиву объектов со значениями, которые возвращает `getProduct` и `getQuantity`.

```js
return getAddedIds(state.cart).map(id => Object.assign(
  {},
  getProduct(state.products, id),
  {
    quantity: getQuantity(state.cart, id)
  }
))
```

Оператор расширения позволяет нам упрощать вызов `map`:

```js
return getAddedIds(state.cart).map(id => ({
  ...getProduct(state.products, id),
  quantity: getQuantity(state.cart, id)
}))
```

Пока оператор расширения все еще на стадии предложения в ECMAScript, вам требуется использовать транспилер, такой как [Babel](http://babeljs.io/), для использования оператора в продакшн-версии. Вы можете использовать существующий пресет `es2015`, установить [`babel-plugin-transform-object-rest-spread`](http://babeljs.io/docs/plugins/transform-object-rest-spread/) и добавить его отдельно в массив `plugins` в вашем файле `.babelrc`.

```js
{
  "presets": ["es2015"],
  "plugins": ["transform-object-rest-spread"]
}
```

Помните, что это все еще экспериментальная функция языка и она может измениться в будущем. Однако, некоторые большие проекты, такие как [React Native](https://github.com/facebook/react-native), уже широко используют это, так что с уверенностью можно сказать, что будет хорошая автоматическая миграция, если функция изменится.

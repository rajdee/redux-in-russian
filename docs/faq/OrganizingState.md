# Redux FAQ: Организация состояния

## Содержание

- [Могу ли я хранить все мое состояние в Redux? Должен ли я всегда использовать setState() из React?](#organizing-state-only-redux-state)
- [Могу ли я хранить функции, промисы или другие несериализируемые данные в моем хранилище состояния?](#organizing-state-non-serializable)
- [Как мне хранить вложенные или дублирующиеся данные в моем состоянии?](#organizing-state-nested-data)


## Организация состояния

<a id="organizing-state-only-redux-state"></a>
### Могу ли я хранить все мое состояние в Redux? Должен ли я всегда использовать setState() из React?

Не существует “правильного” ответа на этот вопрос. Некоторые пользователи предпочитают хранить все данные в Redux, чтобы поддерживать полностью сериализуемую и контролируемую версию своего приложения на протяжении всего времени. Другие предпочитают выносить некритичные данные (UI состояние), как, например “is this dropdown currently open”, в состояние компонента.

***Лучше использовать состояние компонента***. Это _Ваша_ работа, как разработчика, решать, какие части состояния составляют ваше приложение и где каждая из них хранится. Найдите баланс, который подходит Вам, и реализуйте его.

Ниже описаны некоторые негласные правила для определения тех частей данных, которые должны хранится в Redux-хранилище:

- Остальные части приложения используют эти данные?
- Потребуется ли в дальнейшем возможность создавать данные, основанные на этих данных?
- Эти данные используются для управления несколькими компонентами?
- Важна ли Вам возможность восстанавливать это состояние в какой-то момент времени, т.е. отладка по времени (time travel debugging)?
- Требуется ли кэшировать данные, т.е. использовать то, что уже хранится в состоянии вместо повторного запроса?

Существует несколько пакетов, реализующих различные подходы к хранению данных в Redux-хранилище вместо состояния компонента, таких как: [redux-ui](https://github.com/tonyhb/redux-ui), [redux-component](https://github.com/tomchentw/redux-component), [redux-react-local](https://github.com/threepointone/redux-react-local) и другие. Они также позволяют применять принципы Redux и концепцию редюсера к задачам обновления локального состояни компонента, поддерживая идею `this.setState( (previousState) => reducer(previousState, someAction))`.

#### Дополнительная информация

**Статьи**

- [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)
- [Finding `state`'s place with React and Redux](https://medium.com/@adamrackis/finding-state-s-place-with-react-and-redux-e9a586630172)
- [A Case for setState](https://medium.com/@zackargyle/a-case-for-setstate-1f1c47cd3f73)
- [How to handle state in React: the missing FAQ](https://medium.com/react-ecosystem/how-to-handle-state-in-react-6f2d3cd73a0c)
- [Where to Hold React Component Data: state, store, static, and this](https://medium.freecodecamp.com/where-do-i-belong-a-guide-to-saving-react-component-data-in-state-store-static-and-this-c49b335e2a00)
- [The 5 Types of React Application State](http://jamesknelson.com/5-types-react-application-state/)

**Обсуждения**

- [#159: Investigate using Redux for pseudo-local component state](https://github.com/reactjs/redux/issues/159)
- [#1098: Using Redux in reusable React component](https://github.com/reactjs/redux/issues/1098)
- [#1287: How to choose between Redux's store and React's state?](https://github.com/reactjs/redux/issues/1287)
- [#1385: What are the disadvantages of storing all your state in a single immutable atom?](https://github.com/reactjs/redux/issues/1385)
- [Twitter: Should I keep something in React component state?](https://twitter.com/dan_abramov/status/749710501916139520)
- [Twitter: Using a reducer to update a component](https://twitter.com/dan_abramov/status/736310245945933824)
- [React Forums: Redux and global state vs local state](https://discuss.reactjs.org/t/redux-and-global-state-vs-local-state/4187)
- [Reddit: "When should I put something into my Redux store?"](https://www.reddit.com/r/reactjs/comments/4w04to/when_using_redux_should_all_asynchronous_actions/d63u4o8)
- [Stack Overflow: Why is state all in one place, even state that isn't global?](http://stackoverflow.com/questions/35664594/redux-why-is-state-all-in-one-place-even-state-that-isnt-global)
- [Stack Overflow: Should all component state be kept in Redux store?](http://stackoverflow.com/questions/35328056/react-redux-should-all-component-states-be-kept-in-redux-store)

**Библиотеки**

- [Redux Addons Catalog: Component State](https://github.com/markerikson/redux-ecosystem-links/blob/master/component-state.md)

<a id="organizing-state-non-serializable"></a>
### Могу ли я хранить функции, промисы или другие несериализируемые данные в моем хранилище состояния?

Настоятельно рекомендуется класть в хранилище только простые сериализуемые объекты, массивы и примитивы. *Технически* возможно добавить несериализуемые данные в хранилище, но это может сломать способность сохранять и восстанавливать содержимое хранилища, что сделает невозможным отладку по временной шкале (time-travel debugging).

Если Вы смиритесь с тем, что эти функции возможно не будут работать как положено, то Вы можете спокойно использовать несериализуемые данные в Redux-хранилище. В конечном счете, это _Ваше_ приложение, и как оно будет реализовано — решать Вам. Как и со многими другими вещами в Redux, Вы должны четко понимать, на какие компромиссы идете.

#### Дополнительная информация

**Обсуждения**
- [#1248: Is it ok and possible to store a react component in a reducer?](https://github.com/reactjs/redux/issues/1248)
- [#1279: Have any suggestions for where to put a Map Component in Flux?](https://github.com/reactjs/redux/issues/1279)
- [#1390: Component Loading](https://github.com/reactjs/redux/issues/1390)
- [#1407: Just sharing a great base class](https://github.com/reactjs/redux/issues/1407)
- [#1793: React Elements in Redux State](https://github.com/reactjs/redux/issues/1793)


<a id="organizing-state-nested-data"></a>
### Как мне хранить вложенные или дублирующиеся данные в моем состоянии?

Данные с идентификаторами, вложенностью или отношениями, как правило, следует хранить в “нормализированном” стиле: каждый объект должен быть сохранен однажды, идентифицирован, и другие ссылающиеся на него объекты должны хранить только идентификатор, а не копировать весь объект.

Это помогает думать о частях приложения как о базе данных с отдельными “таблицами” на каждый элемент. Такие библиотеки, как [normalizr](https://github.com/gaearon/normalizr) и [redux-orm](https://github.com/tommikaikkonen/redux-orm) могут помочь и облегчить управление нормализированными данными.

#### Дополнительная информация

**Документация**
- [Продвинутое использование: Асинхронные действия](/docs/advanced/AsyncActions.md)
- [Примеры: Real World](/docs/introduction/Examples.html#real-world)
- [Рецепты: Структурирование редюсеров — Предварительные концепциии](/docs/recipes/reducers/PrerequisiteConcepts.md#normalizing-data)
- [Рецепты: Структурирование редюсеров — Нормализация состояния](/docs/recipes/reducers/NormalizingStateShape.md)
- [Примеры: Tree View](https://github.com/reactjs/redux/tree/master/examples/tree-view)

**Статьи**
- [High-Performance Redux](http://somebody32.github.io/high-performance-redux/)
- [Querying a Redux Store](https://medium.com/@adamrackis/querying-a-redux-store-37db8c7f3b0f)

**Обсуждения**
- [#316: How to create nested reducers?](https://github.com/reactjs/redux/issues/316)
- [#815: Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [#946: Best way to update related state fields with split reducers?](https://github.com/reactjs/redux/issues/946)
- [#994: How to cut the boilerplate when updating nested entities?](https://github.com/reactjs/redux/issues/994)
- [#1255: Normalizr usage with nested objects in React/Redux](https://github.com/reactjs/redux/issues/1255)
- [#1269: Add tree view example](https://github.com/reactjs/redux/pull/1269)
- [#1824: Normalising state and garbage collection](https://github.com/reactjs/redux/issues/1824#issuecomment-228585904)
- [Twitter: state shape should be normalized](https://twitter.com/dan_abramov/status/715507260244496384)
- [Stack Overflow: How to handle tree-shaped entities in Redux reducers?](http://stackoverflow.com/questions/32798193/how-to-handle-tree-shaped-entities-in-redux-reducers)
- [Stack Overflow: How to optimize small updates to props of nested components in React + Redux?](http://stackoverflow.com/questions/37264415/how-to-optimize-small-updates-to-props-of-nested-component-in-react-redux)
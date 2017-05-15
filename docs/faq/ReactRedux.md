# Redux FAQ: React Redux

## Содержание

- [Почему мой компонент не перерендеривается? Почему не работает mapStateToProps?](#react-not-rerendering)
- [Почему мой компонент перерендеривается слишком часто?](#react-rendering-too-often)
- [Как я могу ускорить mapStateToProps?](#react-mapstate-speed)
- [Почему у меня недоступен this.props.dispatch в моем подсоединенном компоненте?](#react-props-dispatch)
- [Должен ли я подключить(connect) только мой корневой компонент или я могу подключить несколько компонентов в моем дереве?](#react-multiple-components)

## React Redux

<a id="react-not-rerendering"></a>
### Почему мой компонент не перерендеривается? Почему не работает mapStateToProps?

Случайные мутации или изменения состояния напрямую являются наиболее частыми причинами того, что компонент не перерисоваывается после отправки действия. Redux ожидает, что Ваши редюсеры будут обновлять свои состояни “иммутабельно”, что фактически означает, что надо делать копию данных и изменять уже копию. Если Вы возвращаете тот же самый объект в редюсере, Redux полагает, что ничего не изменилось, даже если Вы изменили содержимое объекта. Аналогично React Redux старается улучшить производительность, делая поверхностное сравнение ссылок входных параметров в `shouldComponentUpdate`, и, если все ссылки остались такими же, возвращает `false`, пропуская обновление Вашего компонента.

Важно помнить, что всякий раз, когда вы обновляете вложенные значения, Вы должны также вернуть новые копии этих значений в Ваше дерево состояний. Если у Вас `state.a.b.c.d`, и Вы хотите обновить данные в `d`, Вам также надо будет вернуть новые копии `c`, `b`, `a`, и `state`. [state tree mutation diagram](http://arqex.com/wp-content/uploads/2015/02/trees.png) показывает, как глубокое изменение в дереве меняет весь путь.

Важно понимать, что “иммутабельное обновление данных” *не означает*, что Вы должны использовать [Immutable.js](https://facebook.github.io/immutable-js/), хотя это конечно вариант. Вы можете делать иммутабельные обновления простых JS-объектов и массивов, используя несколько различных подходов:

- копирование объектов с использование таких функций, как `Object.assign()` и `_.extend()`, а для массивов — `slice()` и `concat()`,
- использование spread-оператора (...) из ES6, который предполагается использовать в следующей версии JavaScript,
- библиотеки, которые оборачивают логику иммутабельного обновления в простые функции.

#### Дополнительная информация

**Документация**

- [Поиск неисправностей](/docs/Troubleshooting.md)
- [React Redux: Поиск неисправностей](https://github.com/reactjs/react-redux/blob/master/docs/troubleshooting.md)
- [Рецепты: Использование оператора расширения](/docs/recipes/UsingObjectSpreadOperator.md)
- [Рецепты: Структурирование редюсеров - Предварительные концепциии](/docs/recipes/reducers/PrerequisiteConcepts.md)
- [Рецепты: Структурирование редюсеров - Паттерны иммутабельного обновления](/docs/recipes/reducers/ImmutableUpdatePatterns.md)

**Статьи**

- [Pros and Cons of Using Immutability with React](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/)
- [React/Redux Links: Immutable Data](https://github.com/markerikson/react-redux-links/blob/master/immutable-data.md)

**Обсуждения**

- [#1262: Immutable data + bad performance](https://github.com/reactjs/redux/issues/1262)
- [React Redux #235: Predicate function for updating component](https://github.com/reactjs/react-redux/issues/235)
- [React Redux #291: Should mapStateToProps be called every time an action is dispatched?](https://github.com/reactjs/react-redux/issues/291)
- [Stack Overflow: Cleaner/shorter way to update nested state in Redux?](http://stackoverflow.com/questions/35592078/cleaner-shorter-way-to-update-nested-state-in-redux)
- [Gist: state mutations](https://gist.github.com/amcdnl/7d93c0c67a9a44fe5761#gistcomment-1706579)


<a id="react-rendering-too-often"></a>
### Почему мой компонент перерендеривается слишком часто?

React Redux реализует несколько оптимизаций, чтобы обеспечить перерисовоку компонента только тогда, когда это действительно необходимо. Одна из них — поверхностное сравнение объединенных в объект параметров функции `mapStateToProps` и аргументов `mapDispatchToProps`, переданных в `connect`. К несчастью, поверхностное сравнение не помогает в случаях, когда новый массив или ссылка на объект создаются при каждом вызове . Типичным примером может быть сопоставление массива идентификаторов с соответствующими ссылками на объекты, как, например:

```js
const mapStateToProps = (state) => {
  return {
    objects: state.objectIds.map(id => state.objects[id])
  }
}
```

Хотя массив может каждый раз хранить в точности те же ссылки, сам массив имеет другую ссылку, а в этом случае поверхностное сравнение возвращает `false`, и React Redux запускает перерисовку компонента-обертки.

Излишние перерисовки могут быть решены сохранением массива объектов в состоянии с использованием редюсера, с кэшированием сопоставляемого массива с использованием [Reselect](https://github.com/reactjs/reselect), или реализацией `shouldComponentUpdate` внутри компонента вручную и выполнением более глубокого сравнения параметров с использованием таких функций, как `_.isEqual`. Будьте осторожны, чтобы не сделать `shouldComponentUpdate()` более затратным, чем сам рендеринг! Всегда используйте порфилировщик для проверки Ваших предположений касаемо производительности.

Для неподключенных компонентов Вы можете проверять, какие параметры передаются. Общая проблема — это наличие в родительском компоненте привязки функции к контексту внутри рендера, как, например, `<Child onClick={this.handleClick.bind(this)} />`. Это создает новую функциональную зависимость каждый раз, когда родитель перерисовывается. Считается хорошей практикой привязывать функции к контексту один раз в конструкторе компонента-родителя.

#### Дополнительная информация

**Документация**

- [FAQ: Производительность - Масштабируемость](/docs/faq/Performance.md#performance-scaling)

**Статьи**

- [A Deep Dive into React Perf Debugging](http://benchling.engineering/deep-dive-react-perf-debugging/)
- [React.js pure render performance anti-pattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)
- [Improving React and Redux Performance with Reselect](http://blog.rangle.io/react-and-redux-performance-with-reselect/)
- [Encapsulating the Redux State Tree](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)
- [React/Redux Links: React/Redux Performance](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md)

**Обсуждения**

- [Stack Overflow: Can a React Redux app scale as well as Backbone?](http://stackoverflow.com/questions/34782249/can-a-react-redux-app-really-scale-as-well-as-say-backbone-even-with-reselect)

**Библиотеки**

- [Redux Addons Catalog: DevTools - Component Update Monitoring](https://github.com/markerikson/redux-ecosystem-links/blob/master/devtools.md#component-update-monitoring)



<a id="react-mapstate-speed"></a>
### Как я могу ускорить мой `mapStateToProps`?

Пока React Redux делает работу по минимизации числа вызовов функции `mapStateToProps`, все еще надо быть уверенным, что Ваш `mapStateToProps` запускается быстро и также минимизирует количество выполняемой работы. Общий рекомендуемый подход — это создавать мемоизированные селекторы функций, используя [Reselect](https://github.com/reactjs/reselect). Эти селекторы могут быть объединены и составлены вместе, селекторы позже будут запущены только в том случае, если входные данные изменились. Это значит, что Вы можете создать селекторы, которые делают такие вещи, как фильтрация или сортировка, и быть уверенными, что на самом деле они будут работать только когда это нужно.

#### Дополнительная информация

**Документация**

- [Рецепты: Оперирование полученными данными](/docs/recipes/ComputingDerivedData.md)

**Статьи**

- [Improving React and Redux Performance with Reselect](http://blog.rangle.io/react-and-redux-performance-with-reselect/)

**Обсуждения**

- [#815: Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [Reselect #47: Memoizing Hierarchical Selectors](https://github.com/reactjs/reselect/issues/47)


<a id="react-props-dispatch"></a>
### Почему у меня недоступен `this.props.dispatch` в моем подсоединенном компоненте?

Функция `connect()` получает 2 основных аргумента, оба необязательных. Первый — `mapStateToProps` — функция, которую Вы предоставляете для вытягивания данных из хранилища, при их изменении, и передачи этих значений в качестве агрументов в Ваш компонент. Вторая — `mapDispatchToProps` — функция, которую Вы предоставляете для возможности использовать `dispatch` функцию хранилища, обычно создавая предварительно связанные (pre-bound) версии генераторов действий, которые будут автоматически отправлять свои действия как только будут вызваны.

Если Вы не передаете Вашу собственную функцию `mapDispatchToProps` при вызове `connect()`, React Redux передаст версию по умолчанию, которая просто возвращает `dispatch` функцию как параметр. Это означает, что если Вы *передаете* свою функцию, то `dispatch` автоматически *не* передается. Если Вы все еще хотите получить `dispatch` как аргумент, Вам надо явно возвращать его самим в Вашей реализации `mapDispatchToProps`.

#### Дополнительная информация

**Документация**

- [React Redux API: connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)

**Обсуждения**

- [React Redux #89: can i wrap multi actionCreators into one props with name?](https://github.com/reactjs/react-redux/issues/89)
- [React Redux #145: consider always passing down dispatch regardless of what mapDispatchToProps does](https://github.com/reactjs/react-redux/issues/145)
- [React Redux #255: this.props.dispatch is undefined if using mapDispatchToProps](https://github.com/reactjs/react-redux/issues/255)
- [Stack Overflow: How to get simple dispatch from this.props using connect w/ Redux?](http://stackoverflow.com/questions/34458261/how-to-get-simple-dispatch-from-this-props-using-connect-w-redux/34458710])


<a id="react-multiple-components"></a>
### Должен ли я подключать (connect) только мой корневой компонент или я могу подключить несколько компонентов в моем дереве?

Раньше документация Redux советовала Вам иметь несколько подключаемых компонентов рядом с корневым. Однако, время и опыт показали, что это, как правильно, требует от нескольких компонентов знать слишком много о требованиях к данным всех их потомков, и заставляет их передавать запутывающее количество параметров.

Текущаяя предложенная лучшая практика — это разделять Ваши компоненты на “представления” и “контейнеры” и извлекать подключенный контейнер компонента везде, где это имеет смысл:

> Акцент на том, что “один компонент-контейнер в качестве корневого”, в примерах Redux был ошибкой. Не используйте это как афоризм. Пытайтесь хранить представления ваших компонентов отдельно. Создавайте контейнеры, подключая их когда это удобно. Всякий раз, когда Вы чувствуете, что Вы дублируете код родительского компонента, чтобы передать данные нескольким потомкам, пора извлечь контейнер. Как правило, как только вы чувствуете, что родитель знает слишком много о “личных” данных или действиях своих детей, пора извлечь контейнер.

По факту, тесты показывают, что большее число подключенных компонентов, как правило, приводит к улучшению производительности, чем меньшее их количество.

В целом, старайтесь найти баланс между пониманием потока данных и областями ответственности Ваших компонентов.

#### Дополнительная информация

**Документация**

- [Basics: Usage with React](/docs/basics/UsageWithReact.md)
- [FAQ: Performance - Scaling](/docs/faq/Performance.md#performance-scaling)

**Статьи**

- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [High-Performance Redux](http://somebody32.github.io/high-performance-redux/)
- [React/Redux Links: Architecture - Redux Architecture](https://github.com/markerikson/react-redux-links/blob/master/react-redux-architecture.md#redux-architecture)
- [React/Redux Links: Performance - Redux Performance](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md#redux-performance)

**Обсуждения**

- [Twitter: emphasizing “one container” was a mistake](https://twitter.com/dan_abramov/status/668585589609005056)
- [#419: Recommended usage of connect](https://github.com/reactjs/redux/issues/419)
- [#756: container vs component?](https://github.com/reactjs/redux/issues/756)
- [#1176: Redux+React with only stateless components](https://github.com/reactjs/redux/issues/1176)
- [Stack Overflow: can a dumb component use a Redux container?](http://stackoverflow.com/questions/34992247/can-a-dumb-component-use-render-redux-container-component)
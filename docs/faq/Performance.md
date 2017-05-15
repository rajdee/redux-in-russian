# Redux FAQ: Производительность

## Содержание

- [Насколько хорошо “масштабируется” Redux с точки зрения производительности и архитектуры?](#performance-scaling)
- [Не будет ли вызов “всех моих редюсеров” для каждого действия медленным?](#performance-all-reducers)
- [Должен ли я иметь полноценный клон моего состояния в редюсере? Не будет ли копирование моего состояния медленным?](#performance-clone-state)
- [Как мне уменьшить количество событий обновления хранилища?](#performance-update-events)
- [Будут ли проблемы с памятью из-за использования “одного дерева состояния”? Будет ли вызов большого количества действий занимать память?](#performance-state-memory)

## Производительность

<a id="performance-scaling"></a>
### Насколько хорошо “масштабируется” Redux с точки зрения производительности и архитектуры?

Пока еще нет окончательного ответа на этот вопрос, в большинстве случаев это не должно быть проблемой.

Работа Redux как правило, делится на несколько частей:
* обработка действие в миддлвэрах и редюсерах (включая дублирование объекта для постоянных обновлений),
* уведомление подписчиков после отправки действий,
* обновление UI-компонентов на основе изменения состояния.

Хотя, конечно, каждый из этих пунктов *может* ухудшить производительность в достаточно сложных ситуациях, но в реализации нет Redux изначально нет ничего медлительного или неэффективного. По факту, React Redux сильно оптимизирован в части избавлениях от ненужных перерисовок, а React-Redux v5 показывает заметные улучшение по сравнению с более поздними версиями.

Redux может быть не так эффективен из коробки, по сравнению с другими библиотеками. Для максимальной производительности отрисовки в React-приложении, состояние должно храниться в нормализованной форме, множество компонентов должны подключаться к одному хранилищу, а не к нескольким, и подключенный список компонентов должен передавать идентификатор каждого элемента своим подключенным потомкам (позволяя тем самым элементам списка искать их собственные данные по ID). Это минимализирует общее количество перерисовок. Использование [мемоизированных](https://ru.wikipedia.org/wiki/%D0%9C%D0%B5%D0%BC%D0%BE%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F) функций также является важным фактором эффективности.

С точки зрения архитектуры, неофициальные данные показывают, что Redux хорошо работает с различными проектами и размерами команды. В настоящее время Redux используется сотнями компаний и тысячами разработчиков, имеет несколько сотен тысяч ежемесячных установок из NPM. Один разработчик сообщил:

> для сравнения, у нас есть ~500 типов действий, ~400 редюсеров, ~150 компонентов, 5 миддлвэров, ~200 действий, ~2300 тестов

#### Дополнительная информация

**Документация**

- [Рецепты: Структурирование редюсеров — Нормализация состояния](/docs/recipes/reducers/NormalizingStateShape.md)

**Статьи**

- [How to Scale React Applications](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/) (accompanying talk: [Scaling React Applications](https://vimeo.com/168648012))
- [High-Performance Redux](http://somebody32.github.io/high-performance-redux/)
- [Improving React and Redux Perf with Reselect](http://blog.rangle.io/react-and-redux-performance-with-reselect/)
- [Encapsulating the Redux State Tree](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)
- [React/Redux Links: Performance - Redux](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md#redux-performance)

**Обсуждения**

- [#310: Who uses Redux?](https://github.com/reactjs/redux/issues/310)
- [#1751: Performance issues with large collections](https://github.com/reactjs/redux/issues/1751)
- [React Redux #269: Connect could be used with a custom subscribe method](https://github.com/reactjs/react-redux/issues/269)
- [React Redux #407: Rewrite connect to offer an advanced API](https://github.com/reactjs/react-redux/issues/407)
- [React Redux #416: Rewrite connect for better performance and extensibility](https://github.com/reactjs/react-redux/pull/416)
- [Redux vs MobX TodoMVC Benchmark: #1](https://github.com/mweststrate/redux-todomvc/pull/1)
- [Reddit: What's the best place to keep the initial state?](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)
- [Reddit: Help designing Redux state for a single page app](https://www.reddit.com/r/reactjs/comments/48k852/help_designing_redux_state_for_a_single_page/)
- [Reddit: Redux performance issues with a large state object?](https://www.reddit.com/r/reactjs/comments/41wdqn/redux_performance_issues_with_a_large_state_object/)
- [Reddit: React/Redux for Ultra Large Scale apps](https://www.reddit.com/r/javascript/comments/49box8/reactredux_for_ultra_large_scale_apps/)
- [Twitter: Redux scaling](https://twitter.com/NickPresta/status/684058236828266496)
- [Twitter: Redux vs MobX benchmark graph - Redux state shape matters](https://twitter.com/dan_abramov/status/720219615041859584)
- [Stack Overflow: How to optimize small updates to props of nested components?](http://stackoverflow.com/questions/37264415/how-to-optimize-small-updates-to-props-of-nested-component-in-react-redux)
- [Chat log: React/Redux perf - updating a 10K-item Todo list](https://gist.github.com/markerikson/53735e4eb151bc228d6685eab00f5f85)
- [Chat log: React/Redux perf - single connection vs many connections](https://gist.github.com/markerikson/6056565dd65d1232784bf42b65f8b2ad)


<a id="performance-all-reducers"></a>
### Не будет ли вызов “всех моих редюсеров” для каждого действия медленным?

Важно иметь ввиду, что Redux-хранилище на самом деле имеет всего одну функцию-редюсер. Хранилище передает текующее состояние и отправленное действие этому единственному редюсеру и позволяет ему обрабатывать это все как надо.

Очевидно, попытка обрабатывать каждое возможное действие в одной функции плохо масштабируется, хотя бы с точки зрения размера функции и читабельности. Таким образом, есть смысл разделить реальную работу на отдельные функции, которые могут быть вызваны редюсером верхнего порядка. В частности, общий рекомендуемый подход — это иметь отдельную функцию (подредюсер), которая ответственна за управление обновлениями на определенном участке состояния в определенном ключе. Функция `combineReducers()`, поставляемая с Redux, — 1 из множества способов реализации этого подхода. Также настоятельно рекомендуется держать состояние хранилища единообразным и нормализированным на столько, на сколько это возможно. В конечном счете, только Вы отвечаете за организацию логики Вашего редюсера любыми способами, какими только пожелаете.

Однако, даже если у Вас много различных редюсеров, объединенных вместе, даже с глубоко вложенным состоянием, скорость редюсера вряд ли будет проблемой. Движки JavaScript способны запускать очень большое количество вызовов функций в секунду, а большинство Ваших редюсеров вероятнее всего используют конструкцию `switch` и возвращают существующее состояние по умолчанию в ответ на большинство действий.

Если вы действительно обеспокоены производительностью редюсера, вы можете использовать такие утилиты, как [redux-ignore](https://github.com/omnidan/redux-ignore) или [reduxr-scoped-reducer](https://github.com/chrisdavies/reduxr-scoped-reducer), чтоб гарантировать, что только определенные редюсеры прослушивают конкретные действия. Также Вы можете использовать [redux-log-slow-reducers](https://github.com/michaelcontento/redux-log-slow-reducers), чтобы провести сравнительный анализ эффективности.

#### Дополнительная информация

**Обсуждения**

- [#912: Proposal: action filter utility](https://github.com/reactjs/redux/issues/912)
- [#1303: Redux Performance with Large Store and frequent updates](https://github.com/reactjs/redux/issues/1303)
- [Stack Overflow: State in Redux app has the name of the reducer](http://stackoverflow.com/questions/35667775/state-in-redux-react-app-has-a-property-with-the-name-of-the-reducer/35674297)
- [Stack Overflow: How does Redux deal with deeply nested models?](http://stackoverflow.com/questions/34494866/how-does-redux-deals-with-deeply-nested-models/34495397)


<a id="performance-clone-state"></a>
### Должен ли я иметь полноценный клон моего состояния в редюсере? Не будет ли копирование моего состояния медленным?

Иммутабельное обновление состояния в большинстве случаев подразумевает создание поверхностных, неглубоких копий. Поверхностные копии более быстрые, чем полноценные, потому что предполагают копирование небольших объектов и полей, и это эффективно снижает перемещение нескольких указателей.

Однако, Вам *надо* создавать скопированные и обновленные объекты для каждого затронутого уровня вложенности. Хотя это не должно быть особенно затратно, но это еще одна причина, почему Вам следует держать Ваше состояние нормализованным и поверхностным на сколько возможно.

> Распростораненное заблуждение Redux: Вам надо полноценно клонировать состояние. На самом деле: если внутри ничего не меняется, достаточно хранить ссылку!

#### Дополнительная информация

**Документация**

- [Рецепты: Структурирование редюсеров - Предварительные концепциии](/docs/recipes/reducers/PrerequisiteConcepts)
- [Рецепты: Структурирование редюсеров - Паттерны иммутабельного обновления](/docs/recipes/reducers/ImmutableUpdatePatterns.md)

**Обсуждения**

- [#454: Handling big states in reducer](https://github.com/reactjs/redux/issues/454)
- [#758: Why can't state be mutated?](https://github.com/reactjs/redux/issues/758)
- [#994: How to cut the boilerplate when updating nested entities?](https://github.com/reactjs/redux/issues/994)
- [Twitter: common misconception - deep cloning](https://twitter.com/dan_abramov/status/688087202312491008)
- [Cloning Objects in JavaScript](http://www.zsoltnagy.eu/cloning-objects-in-javascript/)


<a id="performance-update-events"></a>
### Как мне уменьшить количество событий обновления хранилища?

Redux уведомляет подписчиков после каждой успешной отправки действия (т.е. действие достигнуло хранилища и было обработано редюсерами). В некоторых случаях, может быть полезно урезать количество вызовов подписчиков, особенно если генератор действий отправляет несколько отдельных действий за раз.

Если Вы используете React, помните, что Вы можете улучшить производительность нескольких синхронных отправок обертыванием их в  `ReactDOM.unstable_batchedUpdates()`, но этот API экспериментален и может быть удален в любом выпуске React, поэтому не полагайтесь на это слишком сильно. Взгляните на аналоги:
* [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) — редюсер высокого порядка, который позволяет Вам отправлять несколько действий так, будто это одно действие, и “распаковывать” их в редюсере,
* [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) — расширитель хранилища, который позволяет Вам вызывать подписчиков для множественных отправок не чаще, чем раз в определенное время (debounce),
* [redux-batch](https://github.com/manaflair/redux-batch) — расширитель хранилища, которых обрабатывает отправку набора действий с уведомлением одного подписчика.

#### Дополнительная информация

**Обсуждения**

- [#125: Strategy for avoiding cascading renders](https://github.com/reactjs/redux/issues/125)
- [#542: Idea: batching actions](https://github.com/reactjs/redux/issues/542)
- [#911: Batching actions](https://github.com/reactjs/redux/issues/911)
- [#1813: Use a loop to support dispatching arrays](https://github.com/reactjs/redux/pull/1813)
- [React Redux #263: Huge performance issue when dispatching hundreds of actions](https://github.com/reactjs/react-redux/issues/263)

**Libraries**

- [Redux Addons Catalog: Store - Change Subscriptions](https://github.com/markerikson/redux-ecosystem-links/blob/master/store.md#store-change-subscriptions)


<a id="performance-state-memory"></a>
### Будут ли проблемы с памятью из-за использования “одного дерева состояния”? Будет ли вызов большого количества действий занимать память?

Во первых, с точки зрения использования ресурсов памяти, Redux не сильно отличается от других JavaScript-библиотек. Единственно отличие — все возможные ссылки на объекты вместе вложены в одно дерево, а не хранятся в различных независимах экземплярах модели, как например в Backbone.

Во-вторых, типичное Redux-приложение, вероятно, отчасти *меньше* использует память, чем такое же Backbone-приложение, из-за того, что Redux поощряет использование простых JavaScript-объектов и массивов вместо создания экземпляров моделей и коллекций.

Наконец, Redux держит в памяти в один момент времени только одно дерево состояния со ссылками. Объекты, которые больше не имеют ссылок в этом дереве, обычно будут собраны сборщиком мусора.

Redux сам не хранит историю действий. Однако, Redux DevTools — хранит, таким образом действия могут быть проиграны заново, но это возможно только в процессе разработки и не используется в продакшн-версии.

#### Дополнительная информация

**Документация**

- [Продвинутое использование: Асинхронные действия](/docs/advanced/AsyncActions.md)

**Обсуждения**

- [Stack Overflow: Is there any way to "commit" the state in Redux to free memory?](http://stackoverflow.com/questions/35627553/is-there-any-way-to-commit-the-state-in-redux-to-free-memory/35634004)
- [Reddit: What's the best place to keep initial state?](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)

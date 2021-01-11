# Redux FAQ: Экшены
## Содержание

- [Экшены](#actions)
  - [Почему тип экшена должен быть строкой или по крайней мере сериализуемым? Почему мои типы экшенов должны быть константами?](#actions-string-constants)
    - [Дополнительная информация](#further-information)
  - [Всегда ли редьюсеры и экшены преобразуются "один к одному"?](#actions-reducer-mappings)
    - [Дополнительная информация](#further-information-1)
  - [Как я могу выполнять "побочные эффекты", такие как AJAX вызовы? Зачем нам нужны вещи типа “генераторов экшенов, “thunks” или “мидлвар” для осуществления асинхронного поведения?](#actions-side-effects)
    - [Дополнительная информация](#further-information-2)
  - [Какой асинхронный мидлвар должен я использовать? Как вы выбираете между thunks, sagas, observables или что-то еще?](#what-async-middleware-should-i-use-how-do-you-decide-between-thunks-sagas-observables-or-something-else)
  - [Должен ли я отправлять несколько экшенов подряд от одного генератора экшенов?](#actions-multiple-actions)
    - [Дополнительная информация](#further-information-3)


<a id="actions"></a>
## Экшены

<a id="actions-string-constants"></a>
### Почему `type` должен быть строкой или по крайней мере сериализуемым? Почему мои типы экшенов должны быть константами?

Как и состояние, серилизуемые экшены позволяют использовать несколько определяющих Redux особенностей, таких как отладка с помощью путешествия во времени (time travel) и запись и воспроизведение экшенов. Использование чего-то вроде `Symbol` как типа значений или `instanceof` для проверки типа экшенов сломает это. Строки — сериализуемы и хорошо себя документируют, поэтому они являются наилучшим типом для экшенов. Важно помнить, что *можно* использовать типы Symbols, Promises, или другие несериализуемые значения в экшенах, если экшены предназначены для мидлваров. Экшены должны быть сериализуемыми только в тех случаях, когда они фактически взаимодействуют со стором и передаются редьюсерам.

Мы не можем полостью обеспечить сериализуемые экшены по причинам производительности, поэтому Redux только проверяет, что каждый экшен — это простой объект и его `тип` определен. Остальное зависит от вас, но вы можете заметить, что использование сериализуемых типов помогает в отладке и воспроизведении проблем.

Инкапсуляция и централизация частоиспользуемых кусков кода является ключевым понятием в программировании. Конечно, пока возможно вручную создавать объекты экшенов где угодно и вручную писать каждый тип значения, но определение многоразовых констант делает поддержку кода легче. Если Вы вынесите константы в отдельный файл, то сможете [проверять Ваш `import` на опечатки](https://www.npmjs.com/package/eslint-plugin-import), таким образом вы не сможете случайно использовать неправильные строки.

<a id="further-information"></a>
#### Дополнительная информация

**Документация**

- [Рецепты: Упрощенный шаблон](/docs/recipes/ReducingBoilerplate.md#actions)

**Обсуждения**

- [#384: Recommend that Action constants be named in the past tense](https://github.com/reactjs/redux/issues/384)
- [#628: Solution for simple action creation with less boilerplate](https://github.com/reactjs/redux/issues/628)
- [#1024: Proposal: Declarative reducers](https://github.com/reactjs/redux/issues/1024)
- [#1167: Reducer without switch](https://github.com/reactjs/redux/issues/1167)
- [Stack Overflow: Why do you need 'Actions' as data in Redux?](http://stackoverflow.com/q/34759047/62937)
- [Stack Overflow: What is the point of the constants in Redux?](http://stackoverflow.com/q/34965856/62937)

<a id="actions-reducer-mappings"></a>
### Всегда ли редьюсеры и экшены преобразуются "один к одному"?

Нет. Мы предлагаем вам писать маленькие независимые функции-редьюсеры, которые отвечают за обновление отдельных частей состояния. Мы называем этот подход “композиция редьюсеров”. При этом экшен может быть обработано всеми, какими-то или ни одним из них. Это позволяет компонентам существовать отдельно от фактического изменения данных, так один экшен может затрагивать разные части дерева состояния, и компоненту не нужно знать об этом. Некоторые пользователи предпочитают связывать их вместе, как в принципе “ducks”, но, скорее всего, не "один к одному". Вы должны прекратить использовать эту парадигму, как только чувствуете, что хотите обрабатывать экшен в нескольких редьюсерах.



<a id="further-information-1"></a>
#### Дополнительная информация

**Документация**

- [Основы: Редьюсеры](/docs/basics/Reducers.md)
- [Рецепты: Структурирование редьюсеров](/docs/recipes/StructuringReducers.md)


**Обсуждения**

- [Twitter: most common Redux misconception](https://twitter.com/dan_abramov/status/682923564006248448)
- [#1167: Reducer without switch](https://github.com/reactjs/redux/issues/1167)
- [Reduxible #8: Reducers and action creators aren't a one-to-one mapping](https://github.com/reduxible/reduxible/issues/8)
- [Stack Overflow: Can I dispatch multiple actions without Redux Thunk middleware?](http://stackoverflow.com/questions/35493352/can-i-dispatch-multiple-actions-without-redux-thunk-middleware/35642783)

<a id="actions-side-effects"></a>
### Как я могу выполнять "побочные эффекты", такие как AJAX вызовы? Зачем нам нужны вещи типа “генераторов экшенов, “thunks” или “мидлвар” для осуществления асинхронного поведения?

Это длинная и сложная тема, с большим разнообразием мнений о том, как код должен быть организован и какие подходы должны быть использованы.

Любое полноценное веб-приложение должно выполнять сложную логику, обычно включающую асинхронную работу, такую как AJAX запросы. Этот код уже не является чистой функцией. Такое взаимодействие с окружающим миром носит название [“побочные эффекты”](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%B1%D0%BE%D1%87%D0%BD%D1%8B%D0%B9_%D1%8D%D1%84%D1%84%D0%B5%D0%BA%D1%82_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29).

Redux вдохновлен функциональным программированием и из коробки выполнение побочных эффектов в нем не имеет места. В частности, функции редьюсера всегда *должны* быть чистыми функциями типа `(state, action) => newState`. Однако, мидлвары Redux-а позволяют перехватывать экшены и добавлять к ним сложное поведение, включая побочные эффекты.

В целом, Redux предполагает, что код с побочными эффектами должен быть частью процесса создания экшенов. Пока эта идея *может* быть реализуема внутри UI компонента. В большинстве случаев имеет смысл вынести эту логику в переиспользуемую функцию, тогда она может быть вызвана из нескольких мест — иначе говоря, получаем функцию-генератор экшена (action creator).

Самый простой и часто используемый путь сделать это — добавить [Redux Thunk](https://github.com/gaearon/redux-thunk) мидлвар, который позволяет Вам писать генераторы экшенов с более сложной и асинхронной логикой.

Другой широко применяющийся подход — [Redux Saga](https://github.com/yelouafi/redux-saga), который позволяет Вам писать выглядящий более синхронизированным код, используя генераторы и может действовать как “фоновые потоки” или “демоны” в Redux-приложении.

Еще один подход — [Redux Loop](https://github.com/raisemarketplace/redux-loop), который инвертирует процесс, позволяя вашим редьюсерам объявлять побочные эффекты в ответ на изменение состояния и выполнять их отдельно.

Кроме того, *много* других разработанных сообществом библиотек и идей, каждая со своим видинием того, как побочные эффекты должны быть обработаны.


<a id="further-information-2"></a>
#### Дополнительная информация

**Документация**

- [Продвинутое использование: Асинхронные экшены](/docs/advanced/AsyncActions.md)
- [Продвинутое использование: Асинхронные потоки](/docs/advanced/AsyncFlow.md)
- [Продвинутое использование: Мидлвары](/docs/advanced/Middleware.md)


**Статьи**

- [Redux Side-Effects and You](https://medium.com/@fward/redux-side-effects-and-you-66f2e0842fc3)
- [Pure functionality and side effects in Redux](http://blog.hivejs.org/building-the-ui-2/)
- [From Flux to Redux: Async Actions the easy way](http://danmaz74.me/2015/08/19/from-flux-to-redux-async-actions-the-easy-way/)
- [React/Redux Links: "Redux Side Effects" category](https://github.com/markerikson/react-redux-links/blob/master/redux-side-effects.md)
- [Gist: Redux-Thunk examples](https://gist.github.com/markerikson/ea4d0a6ce56ee479fe8b356e099f857e)


**Обсуждения**

- [#291: Trying to put API calls in the right place](https://github.com/reactjs/redux/issues/291)

- [#455: Modeling side effects](https://github.com/reactjs/redux/issues/455)

- [#533: Simpler introduction to async action creators](https://github.com/reactjs/redux/issues/533)

- [#569: Proposal: API for explicit side effects](https://github.com/reactjs/redux/pull/569)

- [#1139: An alternative side effect model based on generators and sagas](https://github.com/reactjs/redux/issues/1139)

- [Stack Overflow: Why do we need middleware for async flow in Redux?](http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux)

- [Stack Overflow: How to dispatch a Redux action with a timeout?](http://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559)

- [Stack Overflow: Where should I put synchronous side effects linked to actions in redux?](http://stackoverflow.com/questions/32982237/where-should-i-put-synchronous-side-effects-linked-to-actions-in-redux/33036344)

- [Stack Overflow: How to handle complex side-effects in Redux?](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux/33036594)

- [Stack Overflow: How to unit test async Redux actions to mock ajax response](http://stackoverflow.com/questions/33011729/how-to-unit-test-async-redux-actions-to-mock-ajax-response/33053465)

- [Stack Overflow: How to fire AJAX calls in response to the state changes with Redux?](http://stackoverflow.com/questions/35262692/how-to-fire-ajax-calls-in-response-to-the-state-changes-with-redux/35675447)

- [Reddit: Help performing Async API calls with Redux-Promise Middleware.](https://www.reddit.com/r/reactjs/comments/469iyc/help_performing_async_api_calls_with_reduxpromise/)

- [Twitter: possible comparison between sagas, loops, and other approaches](https://twitter.com/dan_abramov/status/689639582120415232)

  <a id="what-async-middleware-should-i-use-how-do-you-decide-between-thunks-sagas-observables-or-something-elses"></a>
  ### Какой асинхронный мидлвар должен я использовать? Как вы выбираете между thunks, sagas, observables или что-то еще?

  Существует [достаточно  много мидлвар для обеспечения асинхронных / побочных эффектов](https://github.com/markerikson/redux-ecosystem-links/blob/master/side-effects.md), но наиболее часто используемые следующие [`redux-thunk`](https://github.com/reduxjs/redux-thunk), [`redux-saga`](https://github.com/redux-saga/redux-saga)и [`redux-observable`](https://github.com/redux-observable/redux-observable). Это разные инструменты с разными преимуществами, недостатками и вариантами использования.

  В качестве общего правила:

  - Thunks лучше всего подходят для сложной синхронной логики (особенно кода, который требует доступа ко всему состоянию хранилища Redux) и простой асинхронной логики (как базовые вызовы AJAX). С использованием `async/await` может быть разумно использовать thunk для некоторой более сложной логики, основанной на промисах.
  - Саги лучше всего подходят для сложной асинхронной логики и несвязанного поведение типа «фонового потока», особенно если вам необходимо слушать отправленные экшены (что нельзя сделать с помощью thunks). Они требуют знания  ES6 генераторов  и операторов «эффектов» «redux-saga».
  - Observables решают те же проблемы, что и саги, но полагаются на RxJS для реализации асинхронного поведения. Они требуют знакомства с RxJS API.

  Мы рекомендуем, чтобы большинство пользователей Redux начинали с thunks, а затем добавляли дополнительную библиотеку побочных эффектов, такую как sagas или observables, если их приложение действительно требует обработки для более сложной асинхронной логики.

  Поскольку у саг и observables есть один и тот же вариант использования, приложение обычно использует один или другой, но не оба. Тем не менее, обратите внимание, что **абсолютно нормально использовать, как thunks, так и sagas или observables вместе**, потому что они решают разные проблемы.
  

  **Статьи**

  - [Decembersoft: What is the right way to do asynchronous operations in Redux?](https://decembersoft.com/posts/what-is-the-right-way-to-do-asynchronous-operations-in-redux/)
  - [Decembersoft: Redux-Thunk vs Redux-Saga](https://decembersoft.com/posts/redux-thunk-vs-redux-saga/)
  - [Redux-Thunk vs Redux-Saga: an overview](https://medium.com/@shoshanarosenfield/redux-thunk-vs-redux-saga-93fe82878b2d)
  - [Redux-Saga V.S. Redux-Observable](https://hackmd.io/s/H1xLHUQ8e#side-by-side-comparison)


  **Обсуждения**
  
  - [Reddit: discussion of using thunks and sagas together, and pros and cons of sagas](https://www.reddit.com/r/reactjs/comments/8vglo0/react_developer_map_by_adamgolab/e1nr597/)
  - [Stack Overflow: Pros/cons of using redux-saga with ES6 generators vs redux-thunk with ES2017 async/await](https://stackoverflow.com/questions/34930735/pros-cons-of-using-redux-saga-with-es6-generators-vs-redux-thunk-with-es2017-asy)
  - [Stack Overflow: Why use Redux-Observable over Redux-Saga?](https://stackoverflow.com/questions/40021344/why-use-redux-observable-over-redux-saga/40027778#40027778)


<a id="actions-multiple-actions"></a>
### Должен ли я отправлять несколько экшенов подряд от одного генератора экшенов?

Нет точного правила, как вы должны структурировать свои экшены. Использование такого асинхронного мидлвара, как Redux Thunk, конечно, позволяет выполнять такие операции, как

- отправка нескольких различных, но взаимосвязанных экшенов подряд;
- обработка экшенов, отображающих прогресс AJAX запроса;
- обработка экшенов, основанных на состоянии;
- обработка других экшенов и проверка обновленного состояния сразу же после этого.
  

В целом, спросите себя, эти экшены взаимосвязаны, но самостоятельны или на самом деле должны быть представлены одним экшеном? Делайте то, что имеет смысл для вашей конкретной ситуации, но старайтесь соблюдать баланс между читабельностью редьюсеров и логом экшенов. Например, для экшена, который содержит все новое дерево состояния, можно сделать ваш редьюсер однострочным, но недостаток в том, что теперь вы не можете проследить историю *причин*, по которым изменения произошли, поэтому отладка становится сложнее. С другой стороны, если вы генерируете ваши экшены в цикле, чтобы держать их разделенными, то, если вы захотите ввести новый тип экшена, то оно будет обрабатываться по-другому.


Старайтесь избегать отправки нескольких синхронных экшенов за раз в тех местах, где вы беспокоитесь о производительности. Есть несколько дополнений и подходов, которые также могут дозировать отправку.

<a id="further-information-3"></a>
#### Дополнительная информация

**Документация**

- [FAQ: Производительность - Уменьшение количество обновлений стора](/docs/faq/Performance.md#performance-update-events)


**Статьи**

- [Idiomatic Redux: Thoughts on Thunks, Sagas, Abstraction, and Reusability](http://blog.isquaredsoftware.com/2017/01/idiomatic-redux-thoughts-on-thunks-sagas-abstraction-and-reusability/#multiple-dispatching)

  

**Обсуждения**

- [#597: Valid to dispatch multiple actions from an event handler?](https://github.com/reactjs/redux/issues/597)
- [#959: Multiple actions one dispatch?](https://github.com/reactjs/redux/issues/959)
- [Stack Overflow: Should I use one or several action types to represent this async action?](http://stackoverflow.com/questions/33637740/should-i-use-one-or-several-action-types-to-represent-this-async-action/33816695)
- [Stack Overflow: Do events and actions have a 1:1 relationship in Redux?](http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux/35410524)
- [Stack Overflow: Should actions be handled by reducers to related actions or generated by action creators themselves?](http://stackoverflow.com/questions/33220776/should-actions-like-showing-hiding-loading-screens-be-handled-by-reducers-to-rel/33226443#33226443)
- [Twitter: "Good thread on the problems with Redux Thunk..."](https://twitter.com/dan_abramov/status/800310164792414208)

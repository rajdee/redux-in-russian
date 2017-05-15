# Redux FAQ: Действия

## Содержание

- [Почему тип действия должен быть строкой или по крайней мере сериализуемым? Почему мои типы действий должны быть константами?](#actions-string-constants)
- [Всегда ли редюсеры и действия преобразуются "один к одному"?](#actions-reducer-mappings)
- [Как я могу выполнять "побочные эффекты", такие как AJAX вызовы? Зачем нам нужны вещи типа “генераторов действий”, “thunks” или “мидлвэр” для осуществления асинхронного поведения?](#actions-side-effects)
- [Должен ли я отправлять несколько действий подряд от одного генератора действия?](#actions-multiple-actions)


## Действия

<a id="actions-string-constants"></a>
### Почему тип действия должен быть строкой или по крайней мере сериализуемым? Почему мои типы действий должны быть константами?

Как и состояние, серилизуемые действия позволяют использовать несколько определяющих Redux особенностей, таких как отладка с помощью путешествия во времени (time travel) и запись и воспроизведение действий. Использование чего-то вроде `Symbol` как типа значений или `instanceof` для проверки типа действий сломает это. Строки — сериализуемы и хорошо себя документируют, поэтому они являются наилучшим типом для действий. Важно помнить, что *можно* использовать типы Symbols, Promises, или другие несериализуемые значения в действиях, если действия предназначены для миддлвэров. Действия должны быть сериализуемыми только в тех случаях, когда они фактически взаимодействуют с хранилищем и передаются редюсерам.

Мы не можем полостью обеспечить сериализуемые действия по причинам производительности, поэтому Redux только проверяет, что каждое действие — это простой объект и его тип определен. Остальное зависит от Вас. Но Вы можете заметить, что использование сериализуемых типов помогает в отладке и воспроизведении проблем.

Инкапсуляция и централизация частоиспользуемых кусков кода является ключевым понятием в программировании. Конечно, пока возможно вручную создавать объекты действия где угодно и вручную писать каждый тип значения, но определение многоразовых констант делает поддержку кода легче. Если Вы вынесите константы в отдельный файл, то сможете [проверять Ваш `import` на опечатки](https://www.npmjs.com/package/eslint-plugin-import), таким образом вы не сможете случайно использовать неправильные строки.

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
### Всегда ли редюсеры и действия преобразуются "один к одному"?

Нет. Мы предлагаем Вам писать маленькие независимые функции-редюсеры, которые отвечают за обновление отдельных частей состояния. Мы называем этот подход “композиция редюсеров”. При этом действие может быть обработано всеми, какими-то или ни одним из них. Это позволяет компонентам существовать отдельно от фактического изменения данных, так одно действие может затрагивать разные части дерева состояния, и компоненту не нужно знать об этом. Некоторые пользователи предпочитают связывать их вместе, как в принципе “ducks”, но, скорее всего, не "один к одному". Вы должны прекратить использовать эту парадигму, как только чувствуете, что хотите обрабатывать действие в нескольких редюсерах.

#### Дополнительная информация

**Документация**

- [Основы: Редюсеры](/docs/basics/Reducers.md)
- [Рецепты: Структурирование редюсеров](/docs/recipes/StructuringReducers.md)

**Обсуждения**

- [Twitter: most common Redux misconception](https://twitter.com/dan_abramov/status/682923564006248448)
- [#1167: Reducer without switch](https://github.com/reactjs/redux/issues/1167)
- [Reduxible #8: Reducers and action creators aren't a one-to-one mapping](https://github.com/reduxible/reduxible/issues/8)
- [Stack Overflow: Can I dispatch multiple actions without Redux Thunk middleware?](http://stackoverflow.com/questions/35493352/can-i-dispatch-multiple-actions-without-redux-thunk-middleware/35642783)


<a id="actions-side-effects"></a>
### Как я могу выполнять "побочные эффекты", такие как AJAX вызовы? Зачем нам нужны вещи типа “генераторов действий”, “thunks” или “мидлвэр” для осуществления асинхронного поведения?

Это длинная и сложная тема, с большим разнообразием мнений о том, как код должен быть организован и какие подходы должны быть использованы.

Любое полноценное веб-приложение должно выполнять сложную логику, обычно включающую асинхронную работу, такую как AJAX запросы. Этот код уже не является чистой функцией. Такое взаимодействие с окружающим миром носит название [“побочные эффекты”](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%B1%D0%BE%D1%87%D0%BD%D1%8B%D0%B9_%D1%8D%D1%84%D1%84%D0%B5%D0%BA%D1%82_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29).

Redux вдохновлен функциональным программированием и из коробки выполнение побочных эффектов в нем не имеет места. В частности, функции редюсера всегда *должны* быть чистыми функциями типа `(state, action) => newState`. Однако, миддлвэры Redux-а позволяют перехватывать действия и добавлять к ним сложное поведение, включая побочные эффекты.

В целом, Redux предполагает, что код с побочными эффектами должен быть частью процесса создания действий. Пока эта идея *может* быть реализуема внутри UI компонента. В большинстве случаев имеет смысл вынести эту логику в переиспользуемую функцию, тогда она может быть вызвана из нескольких мест — иначе говоря, получаем функцию-генератор действия (action creator).

Самый простой и часто используемый путь сделать это — добавить [Redux Thunk](https://github.com/gaearon/redux-thunk) миддлвэр, который позволяет Вам писать генераторы действий с более сложной и асинхронной логикой.

Другой широко применяющийся подход — [Redux Saga](https://github.com/yelouafi/redux-saga), который позволяет Вам писать выглядящий более синхронизированным код, используя генераторы, и может действовать как “фоновые потоки” или “демоны” в Redux-приложении.

Еще один подход — [Redux Loop](https://github.com/raisemarketplace/redux-loop), который инвертирует процесс, позволяя вашим редюсерам объявлять побочные эффекты в ответ на изменение состояния и выполнять их отдельно.

Кроме того, *много* других разработанных сообществом библиотек и идей, каждая со своим видинием того, как побочные эффекты должны быть обработаны.


#### Дополнительная информация
**Документация**

- [Продвинутое использование: Асинхронные действия](/docs/advanced/AsyncActions.md)
- [Продвинутое использование: Асинхронные потоки](/docs/advanced/AsyncFlow.md)
- [Продвинутое использование: Миддлвэры (Middleware)](/docs/advanced/Middleware.md)

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


<a id="actions-multiple-actions"></a>
### Должен ли я отправлять несколько действий подряд от одного генератора действия?

Нет точного правила, как Вы должны структурировать свои действия. Использование такого асинхронного миддлвэра, как Redux Thunk, конечно, позволяет выполнять такие операции, как

- отправка нескольких различных, но взаимосвязанных действий подряд,;
- обработка действий, отображающих прогресс AJAX запроса;
- обработка действий, основанных на состоянии;
- обработка других действий и проверка обновленного состояния сразу же после этого.

В целом, спросите себя, эти действия взаимосвязаны, но самостоятельны, или на самом деле должны быть представлены одним действием? Делайте то, что имеет смысл для Вашей конкретной ситуации, но старайтесь соблюдать баланс между читабельностью редюсеров и логом действий. Например, для действия, которое содержит все новое дерево состояния, можно сделать ваш редюсер однострочным, но недостаток в том, что теперь вы не можете проследить историю *причин*, по которым изменения произошли, поэтому отладка становится сложнее. С другой стороны, если Вы генерируете ваши действия в цикле, чтобы держать их разделенными, то, если Вы захотите ввести новый тип действия, то оно будет обрабатываться по-другому.

Старайтесь избегать отправки нескольких синхронных действий за раз в тех местах, где вы беспокоитесь о производительности. Есть несколько дополнений и подходов, которые также могут дозировать отправку.

#### Дополнительная информация

**Документация**

- [FAQ: Производительность - Уменьшение количество обновлений хранилища](/docs/faq/Performance.md#performance-update-events)

**Обсуждения**

- [#597: Valid to dispatch multiple actions from an event handler?](https://github.com/reactjs/redux/issues/597)
- [#959: Multiple actions one dispatch?](https://github.com/reactjs/redux/issues/959)
- [Stack Overflow: Should I use one or several action types to represent this async action?](http://stackoverflow.com/questions/33637740/should-i-use-one-or-several-action-types-to-represent-this-async-action/33816695)
- [Stack Overflow: Do events and actions have a 1:1 relationship in Redux?](http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux/35410524)
- [Stack Overflow: Should actions be handled by reducers to related actions or generated by action creators themselves?](http://stackoverflow.com/questions/33220776/should-actions-like-showing-hiding-loading-screens-be-handled-by-reducers-to-rel/33226443#33226443)

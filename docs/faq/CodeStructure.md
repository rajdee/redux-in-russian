# Redux FAQ: Структура кода

## Содержание

- [Как должна выглядить моя файловая структура? Как я должен группировать генераторы экшенов и редьюсеры в проекте? Где должны быть селекторы?](#structure-file-structure)
- [Как я должен разделять логику между редьюсерами и генераторами экшенов? Где должна быть "бизнес-логика"?](#structure-business-logic)
- [Почему я должен использовать генераторы экшенов?](#why-should-i-use-action-creators)
- [Где должны находиться веб-сокеты и другие постоянные соединения](#where-should-websockets-and-other-persistent-connections-live)


## Структура кода

<a id="structure-file-structure"></a>

### Как должна выглядить моя файловая структура? Как я должен группировать генераторы экшенов и редьюсеры в проекте? Где должны быть селекторы?

Поскольку Redux - это просто библиотека хранения данных, у нее нет точного мнения о том, как ваш проект должен быть структурирован. Тем не менее, есть ряд общих паттернов, которые большинство разработчиков Redux склонны использовать:

- Rails-style: отдельные директории для “экшенов”, “констант”, “редьюсеров”, “контейнеров” и “компонент”
- Domain-style: отдельные директории для фичи или домена, возможно, с поддиректориями для каждого типа файлов
- “Ducks”: похож на domain-style, но явно связывающий экшены и редьюсеры, часто определяя их в том же файле

В целом предполагается, что селекторы определены наряду с редьюсерами и экспортируются, а затем повторно используются в других местах (таких как, функции `mapStateToProps`, асинхронные генераторы экшенов или саги) чтобы сосредоточить весь код, который знает о фактической форме дерева состояния в файлах редьюсеров.

Тогда как, в конечном итоге, не имеет значения, как вы размещаете ваш код на диске, важно помнить, что экшены и редьюсеры не следует рассматривать изолированно. Допускается (и рекомендуется) для редьюсера, быть определенным в одной папке, чтобы реагировать на экшены, определенные в другой папке.

#### Дополнительная информация

**Документация**
- [FAQ: Actions - "1:1 mapping between reducers and actions?"](/docs/faq/Actions.md#actions-reducer-mappings)

**Статьи**
- [How to Scale React Applications](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/) (accompanying talk: [Scaling React Applications](https://vimeo.com/168648012))

- [Redux Best Practices](https://medium.com/lexical-labs-engineering/redux-best-practices-64d59775802e)

- [Rules For Structuring (Redux) Applications ](http://jaysoo.ca/2016/02/28/organizing-redux-application/)

- [A Better File Structure for React/Redux Applications](http://marmelab.com/blog/2015/12/17/react-directory-structure.html)

- [Organizing Large React Applications](http://engineering.kapost.com/2016/01/organizing-large-react-applications/)

- [Four Strategies for Organizing Code](https://medium.com/@msandin/strategies-for-organizing-code-2c9d690b6f33)

- [Encapsulating the Redux State Tree](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)

- [Redux Reducer/Selector Asymmetry](http://randycoulman.com/blog/2016/09/20/redux-reducer-selector-asymmetry/)

- [Modular Reducers and Selectors](http://randycoulman.com/blog/2016/09/27/modular-reducers-and-selectors/)

- [My journey towards a maintainable project structure for React/Redux](https://medium.com/@mmazzarolo/my-journey-toward-a-maintainable-project-structure-for-react-redux-b05dfd999b5)

- [React/Redux Links: Architecture - Project File Structure](https://github.com/markerikson/react-redux-links/blob/master/react-redux-architecture.md#project-file-structure)

  **Обсуждения**

- [#839: Emphasize defining selectors alongside reducers](https://github.com/reduxjs/redux/issues/839)
- [#943: Reducer querying](https://github.com/reduxjs/redux/issues/943)
- [React Boilerplate #27: Application Structure](https://github.com/mxstbr/react-boilerplate/issues/27)
- [Stack Overflow: How to structure Redux components/containers](http://stackoverflow.com/questions/32634320/how-to-structure-redux-components-containers/32921576)
- [Twitter: There is no ultimate file structure for Redux](https://twitter.com/dan_abramov/status/783428282666614784)

<a id="structure-business-logic"></a>

### Как я должен разделять логику между редьюсерами и генераторами экшенов? Где должна быть "бизнес-логика"?

Нет определенного четкого ответа на то, какие части логики следует переносить в редьюсер, а какие в генератор экшенов. Некоторые разработчики предпочитают иметь "толстые" генераторы экшенов, с "тонкими" редьюсерами, которые просто берут данные в экшене (action) и слепо добавляют их в соответствующее состояние.
Другие пытаются подчеркнуть, сохраняя экшенв, как можно маленькими, и свести к минимуму использование `getState()` в генераторе экшенов. (Для целей данного вопроса, другие асинхронные подходы, такие, как саги и обзерваблы, попадают в категорию "генераторы экшенов")

Есть некоторая потенциальная выгода от использования большего количества логики в ваших редьюсерах. Вероятно, что типы экшенов будут более семантическими и более значимыми (например, «USER_UPDATED» вместо «SET_STATE»). Кроме того, наличие большего количества логики в редьюсеров означает, что больше функциональности будет зависеть от отладки путешествия во времени.

Этот комментарий красиво подводит итог этого разделения:

> Теперь проблема в том, что поместить в генератор экшена, а что в редьюсер, выбор между толстыми и тонкими объектами экшенов. Если вы кладете всю логику в генератор экшенов, вы в конечном итоге остаетесь с толстыми объектами экшенов, которые в своей основе декларируют обновления состояния. Редьюсеры становятся чистыми, глупыми, добавлять, удалять, обновлять эти функции. Они будут легко композируемы. Но не так много бизнес-логики будет там.
> Если вы размещаете больше логики в редьюсерах, вы в конечном итоге имеете красивые, тонкие объекты экшенов, большая часть вашей логики данных в одном месте, но ваши редьюсеры сложнее составлять, т.к. вам может понадобиться информация из других веток. Вы в конечном итоге с большими редьюсерами или редьюсерами, которые принимают дополнительные аргументы выше в состоянии.

Найти баланс между этими двумя крайностями и вы освоите Redux.


#### Дополнительная информация

**Статьи**
- [Where do I put my business logic in a React/Redux application?](https://medium.com/@jeffbski/where-do-i-put-my-business-logic-in-a-react-redux-application-9253ef91ce1)
- [How to Scale React Applications](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/)
- [The Tao of Redux, Part 2 - Practice and Philosophy. Thick and thin reducers.](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-2/#thick-and-thin-reducers)

**Обсуждения**
- [How putting too much logic in action creators could affect debugging](https://github.com/reduxjs/redux/issues/384#issuecomment-127393209)

- [#384: The more that's in a reducer, the more you can replay via time travel](https://github.com/reduxjs/redux/issues/384#issuecomment-127393209)

- [#1165: Where to put business logic / validation?](https://github.com/reduxjs/redux/issues/1165)

- [#1171: Recommendations for best practices regarding action-creators, reducers, and selectors](https://github.com/reduxjs/redux/issues/1171)

- [Stack Overflow: Accessing Redux state in an action creator?](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator/35674575)

- [#2796: Gaining clarity on "business logic"](https://github.com/reduxjs/redux/issues/2796#issue-289298280)

- [Twitter: Moving away from unclear terminology...](https://twitter.com/FwardPhoenix/status/952971237004926977)

  <a id="why-should-i-use-action-creators"></a>

  ### Почему я должен использовать генераторы экшенов?

  Redux не требует генераторов экшенов. Вы можете создавать экшены любым удобным для вас способом, включая простую передачу литерала объекта в `dispatch`. Генераторы экшенов пришли из [архитектуры Flux](https://facebook.github.io/react/blog/2014/07/30/flux-actions-and-the-dispatcher.html#actions-and-actioncreators) и были приняты сообществом Redux, потому что они предлагают несколько преимуществ.

  Генераторы экшенов более поддерживаемы. Обновления экшена могут быть сделаны в одном месте и применены везде. Все экземпляры экшена гарантированно имеют одинаковую форму и одинаковые значения по умолчанию.

  Генераторы экшенов тестируемы. Правильность встроенного экшена должна быть проверена вручную. Как и любая функция, тесты для генератора экшена можно написать один раз и запустить автоматически.

  Генераторы экшенов легче документировать. Параметры генератора экшена перечисляют зависимости экшена. А централизация определения экшена обеспечивает удобное место для комментариев к документации. Когда экшены записаны в строке, эту информацию сложнее собирать и передавать.

  Генераторы экшенов - более мощная абстракция. Создание экшена часто включает преобразование данных или выполнение запросов AJAX. Генераторы экшенов предоставляют единый интерфейс для этой разнообразной логики. Эта абстракция упрощает компоненту при отправки экшена, не усложняя деталями его создания.
  

  #### Дальнейшая информация

  **Статьи**

  - [Idiomatic Redux: Why use action creators?](http://blog.isquaredsoftware.com/2016/10/idiomatic-redux-why-use-action-creators/)


  **Обсуждения**

  - [Reddit: Redbox - Redux action creation made simple](https://www.reddit.com/r/reactjs/comments/54k8js/redbox_redux_action_creation_made_simple/d8493z1/?context=4)


  ### Где должны существовать веб-сокеты и другие постоянные соединения?

  Мидлвар - это правильное место для постоянных соединений, типа веб-сокетов в Redux приложении, по ряду причин:

  - Мидлвар существует на протяжении всего жизненного цикла приложения
  - Как и в случае самого стора, вам, вероятно, нужен только один экземпляр данного соединения, который может использовать все приложение
  - Мидлвару доступны все отправленные экшены и сами могу отправлять экшены. Это означает, что мидлвар может выполнять отправленные экшены и превращать их в сообщения, отправляемые через веб-сокет и отправлять новые экшены при получении сообщения через веб-сокет.
  - Экземпляр подключения веб-сокета не сериализуем, поэтому [он не принадлежит самому состоянию стора](/faq/organizing-state#organizing-state-non-serializable)
    

  См. [этот пример, который показывает, как мидлвар сокетов может отправлять и реагировать на экшены Redux](https://gist.github.com/markerikson/3df1cf5abbac57820a20059287b4be58).

  Существует множество мидлваров для веб-сокетов и других подобных соединений - см. ссылку ниже.
  

  **Библиотеки**

  - [Middleware: Socket and Adapters](https://github.com/markerikson/redux-ecosystem-links/blob/master/middleware-sockets-adapters.md)

  

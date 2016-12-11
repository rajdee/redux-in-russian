# Redux FAQ: Структура кода

## Содержание

- [Как должна выглядить моя файловая структура? Как я должен группировать генераторы действий и редюсеры в проекте? Где должны быть селекторы?](#structure-file-structure)
- [Как я должен разделять логику между редюсерами и генераторами действий? Где должна быть "бизнес-логика"?](#structure-business-logic)


## Структура кода

<a id="structure-file-structure"></a>
### Как должна выглядить моя файловая структура? Как я должен группировать генераторы действий и редюсеры в проекте? Где должны быть селекторы?

Поскольку Redux - это просто библиотека хранения данных, у нее нет точного мнения о том, как ваш проект должен быть структурирован. Тем не менее, есть ряд общих паттернов, которые большинство разработчиков Redux склонны использовать:

- Rails-style: отдельные директории для “действий”, “констант”, “редюсеров”, “контейнеров” и “компонент”
- Domain-style: отдельные директории для фичи или домена, возможно, с поддиректориями для каждого типа файлов
- “Ducks”: похож на domain-style, но явно связывающий действия и редюсеры, часто определяя их в том же файле

В целом предполагается, что селекторы определены наряду с редюсерами и экспортируются, а затем повторно используются в других местах (таких как, функции `mapStateToProps`, асинхронные генераторы действий или саги) чтобы сосредоточить весь код, который знает о фактической форме дерева состояния в файлах редюсеров.

Тогда как, в конечном итоге, не имеет значения, как вы размещаете ваш код на диске, важно помнить, что действия и редюсеры не следует рассматривать изолированно. Допускается (и рекомендуется) для редюсера, быть определенным в одной папке, чтобы реагировать на действия, определенные в другой папке.

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
- [#839: Emphasize defining selectors alongside reducers](https://github.com/reactjs/redux/issues/839)
- [#943: Reducer querying](https://github.com/reactjs/redux/issues/943)
- [React Boilerplate #27: Application Structure](https://github.com/mxstbr/react-boilerplate/issues/27)
- [Stack Overflow: How to structure Redux components/containers](http://stackoverflow.com/questions/32634320/how-to-structure-redux-components-containers/32921576)
- [Twitter: There is no ultimate file structure for Redux](https://twitter.com/dan_abramov/status/783428282666614784)


<a id="structure-business-logic"></a>
### Как я должен разделять логику между редюсерами и генераторами действий? Где должна быть "бизнес-логика"?

Нет определенного четкого ответа на то, какие части логики следует переносить в редюсер, а какие в генератор действий. Некоторые разработчики предпочитают иметь "толстые" генераторы действий, с "тонкими" редюсерами, которые просто берут данные в действии (action) и слепо добавляют их в соответствующее состояние.
Другие пытаются подчеркнуть, сохраняя действия, как можно маленькими, и свести к минимуму использование `getState()` в генераторе действий. (Для целей данного вопроса, другие асинхронные подходы, такие, как саги и обзерваблы, попадают в категорию "action creator")

Этот комментарий красиво подводит итог этого разделения:

> Теперь проблема в том, что поместить в генератор действия, а что в редюсер, выбор между толстыми и тонкими объектами действий. Если вы кладете всю логику в генератор действия, вы в конечном итоге остаетесь с толстыми объектами действий, которые в своей основе декларируют обновления состояния. Редюсеры становятся чистыми, глупыми, добавлять, удалять, обновлять эти функции. Они будут легко композируемы. Но не так много бизнес-логики будет там.
> Если вы размещаете больше логики в редюсерах, вы в конечном итоге имеете красивые, тонкие объекты действия, большая часть вашей логики данных в одном месте, но ваши редюсеры сложнее составлять, т.к. вам может понадобиться информация из других веток. Вы в конечном итоге с большими редюсерами или редюсерами, которые принимают дополнительные аргументы выше в состоянии.

Найти баланс между этими двумя крайностями и вы освоите Redux.


#### Дополнительная информация

**Статьи**
- [Where do I put my business logic in a React/Redux application?](https://medium.com/@jeffbski/where-do-i-put-my-business-logic-in-a-react-redux-application-9253ef91ce1)
- [How to Scale React Applications](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/)

**Обсуждения**
- [#1165: Where to put business logic / validation?](https://github.com/reactjs/redux/issues/1165)
- [#1171: Recommendations for best practices regarding action-creators, reducers, and selectors](https://github.com/reactjs/redux/issues/1171 )
- [Stack Overflow: Accessing Redux state in an action creator?](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator/35674575)

# Redux FAQ: Разное

## Содержание

- [Существуют ли большие, "настоящие" проекты на Redux?](#miscellaneous-real-projects)
- [Как мне реализовать аутентификацию в Redux?](#miscellaneous-authentication)

## Разное

<a id="miscellaneous-real-projects"></a>
### Существуют ли большие, "настоящие" проекты на Redux?

Да, множество! Вот несколько примеров:

- [Twitter's mobile site](https://twitter.com/necolas/status/727538799966715904)
- [Wordpress's new admin page](https://github.com/Automattic/wp-calypso)
- [Firefox's new debugger](https://github.com/jlongster/debugger.html)
- [Mozilla's experimental browser testbed](https://github.com/mozilla/tofino)
- [The HyperTerm terminal application](https://github.com/zeit/hyperterm)

И еще очень-очень много! Redux Addons Catalog имеет **[список основанных на Redux приложений и примеров](https://github.com/markerikson/redux-ecosystem-links/blob/master/apps-and-examples.md)**, что указывает на целый ряд реальных приложений, больших и маленьких.

#### Дополнительная информация

**Документация**

- [Введение: Примеры](/docs/introduction/Examples.md)

**Обсуждения**

- [Reddit: Large open source react/redux projects?](https://www.reddit.com/r/reactjs/comments/496db2/large_open_source_reactredux_projects/)
- [HN: Is there any huge web application built using Redux?](https://news.ycombinator.com/item?id=10710240)


<a id="miscellaneous-authentication"></a>
### Как мне реализовать аутентификацию в Redux?

Аутентификация необходима во всех реальных приложениях. Когда разговор идет об аутентификации, Вы должны помнить, что ничего не меняется от Вашей организации приложения, и Вы должны реализовывать аутентификацию тем же путем, что и любой другой функционал. Это относительно просто:

1. Создайте действия для `LOGIN_SUCCESS`, `LOGIN_FAILURE`, и т.д.

2. Создайте генераторы действий, которые будут брать учетные данные, флаг для обозначения успешной аутентификации, и токен или сообщение об ошибке в качестве полезной нагрузки.

3. Создайте асинхронный генератор действий с помощью Redux Thunk миддлвэра или любого другого миддлвэра, который Вы считаете пригодным для отправки запросов по API и который возвращает токен, если данные верны. Затем сохраните токен в локальном хранилище или покажите сообщение пользователю, если запрос неудачный. Вы можете улучшить эти сайд эффекты в генераторе действий, написанном на предыдущем шаге.

4. Создайте редюсер, который возвращает следующее состояние для каждого возможного исхода аутентификации (`LOGIN_SUCCESS`, `LOGIN_FAILURE`, и т.д.).

#### Дополнительная информация

**Статьи**

- [Authentication with JWT by Auth0](https://auth0.com/blog/2016/01/04/secure-your-react-and-redux-app-with-jwt-authentication/)
- [Tips to Handle Authentication in Redux](https://medium.com/@MattiaManzati/tips-to-handle-authentication-in-redux-2-introducing-redux-saga-130d6872fbe7)

**Примеры**

- [react-redux-jwt-auth-example](https://github.com/joshgeller/react-redux-jwt-auth-example)

**Библиотеки**

- [Redux Addons Catalog: Use Cases - Authentication](https://github.com/markerikson/redux-ecosystem-links/blob/master/use-cases.md#authentication)

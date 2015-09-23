# Примеры

Redux распространяется с несколькими примерами в своих [исходных кодах](https://github.com/rackt/redux/tree/master/examples).
**Для того, чтобы запустить любой из них, достаточно склонировать репозиторий и запустить `npm install` в родительской директории и в директории примера.**

>##### Особенности при копировании
>Если вы скопировали примеры Redux из их директорий, удалите эти строки из файлов `webpack.config.js`:
>
>```js
>alias: {
>   'redux': path.join(__dirname, '..', '..', 'src')
>},
>```
>и
>```js
>{
>   test: /\.js$/,
>   loaders: ['babel'],
>   include: path.join(__dirname, '..', '..', 'src')
>},
```
>
> Иначе они будут пытаться вызвать Redux из родительской `src` папки и сборка сломается.

## Счетчик

Запуск примера [счетчика](https://github.com/rackt/redux/tree/master/examples/counter):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/counter
npm install

npm start
open http://localhost:3000/
```

Это описывает:

* Базовый шаблон разработки Redux;
* Тестирование.

## TodoMVC

Запуск [TodoMVC](https://github.com/rackt/redux/tree/master/examples/todomvc):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/todomvc
npm install

npm start
open http://localhost:3000/
```

Это описывает:

* Базовый шаблон разработки Redux с двумя редьюсерами;
* Обновление хранимой информации;
* Тестирование.

## Async

Запуск [Async](https://github.com/rackt/redux/tree/master/examples/async):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/async
npm install

npm start
open http://localhost:3000/
```

Это описывает:

* Базовый шаблон асинхронный разработки Redux с [redux-thunk](https://github.com/gaearon/redux-thunk);
* Кеширование запросов и вывод спиннера, пока данные подгружаются;
* Инвалидирование закэшированных данных.

## Real World

Запуск примера [Real World](https://github.com/rackt/redux/tree/master/examples/real-world):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/real-world
npm install

npm start
open http://localhost:3000/
```

Это описывает:

* Real-world асинхронный Redux шаблон;
* Сохрание объектов в нормализованном кэше объектов;
* Отдельный middleware для вызовов API;
* Кеширование запросов и вывод спиннера, пока данные подгружаются;
* Пагинация;
* Роутинг.

## Больше примеров

Больше примеров вы сможете найти в [Awesome Redux](https://github.com/xgrommx/awesome-redux).

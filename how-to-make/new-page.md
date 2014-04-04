# Как создать новую страницу

Вы уже склонировали `bevis-stub`, запустили в корне проекта команду `make`, увидели,
как качаются какие-то библиотеки (возможно ждали долго, но это только первый раз).

Но вот в консоль перестали сыпаться сообщения^ и последнее, что вы видите, это что-то похожее на это:

```
ENV= node_modules/.bin/enb make
19:16:25.764 - build started
19:16:25.982 - [rebuild] [build/index/index.levels] levels
19:16:25.982 - [rebuild] [build/test/test.levels] levels
19:16:25.982 - [rebuild] [build/index/index.bemdecl.js] file-provider
19:16:25.986 - [rebuild] [build/test/test.bemdecl.js] file-provider
19:16:25.997 - [rebuild] [build/test/test.deps.js] deps-with-modules
19:16:25.998 - [rebuild] [build/test/test.files] files
19:16:25.998 - [rebuild] [build/test/test.dirs] files
19:16:26.085 - [rebuild] [build/test/test.js] js
19:16:26.086 - [rebuild] [build/test/test.css] css-stylus-with-autoprefixer
19:16:26.087 - [rebuild] [build/test/_test.js] file-copy
19:16:26.088 - [rebuild] [build/test/_test.css] file-copy
19:16:26.093 - [rebuild] [build/index/index.deps.js] deps-with-modules
19:16:26.093 - [rebuild] [build/index/index.files] files
19:16:26.093 - [rebuild] [build/index/index.dirs] files
19:16:26.093 - [isValid] [build/index/index.js] js
19:16:26.093 - [isValid] [build/index/index.css] css-stylus-with-autoprefixer
19:16:26.094 - [isValid] [build/index/_index.js] file-copy
19:16:26.094 - [isValid] [build/index/_index.css] file-copy
19:16:26.094 - build finished - 330ms

DEBUG: Running node-supervisor with
DEBUG:   program 'server/boot.js'
DEBUG:   --watch 'server,configs'
DEBUG:   --ignore 'undefined'
DEBUG:   --extensions 'node|js'
DEBUG:   --exec 'node'

DEBUG: Starting child process with 'node server/boot.js'
DEBUG: Watching directory '~/bevis-stub/server' for changes.
DEBUG: Watching directory '~/bevis-stub/configs' for changes.
19:16:26 - info: worker 23454 started
19:16:26 - info: app started on 8080
19:16:26 - info: worker 23455 started
19:16:26 - info: app started on 8080
```

Если всё так, проект собран, сервер запущен (на 8080 порту, но это можно и поменять), и если теперь вы в браузере 
запросите `localhost:8080`, то увидите нашу страничку "Привет, BEViS!".

----

Всё ли так? Если нет, заведите issue, мы вам поможем :)

Не понимаете всего, что написано в консоли? Не важно. Я тоже слабо понимаю ;)

----

Теперь создадим страницу `test`, которая будет отвечать пользователю по адресу `localhost:8080/test/`.


Кстати, сразу проверим, что нам ответит браузер, если мы прямо сейчас сделаем запрос к несуществующей странице. Пишем
 в адресную строку браузера `localhost:8080/test/` и видим ответ "Internal error". Это нормально, так и должно быть :)
 
Проверяем консоль, так и есть - консоль подтверждает, что всё идёт, как задумано:

```
19:27:05 - error: Error: Target not found: build/test/test.page.js
    at Error (<anonymous>)
    at TargetNotFoundError (~/bevis-stub/node_modules/enb/lib/errors/target-not-found-error.js:11:23)
    at module.exports.inherit._resolveTarget (~/bevis-stub/node_modules/enb/lib/make.js:383:15)
    at ~/bevis-stub/node_modules/enb/lib/make.js:400:36
    at Array.forEach (native)
    at module.exports.inherit._resolveTargets (~/bevis-stub/node_modules/enb/lib/make.js:399:21)
    at module.exports.inherit.buildTargets (~/bevis-stub/node_modules/enb/lib/make.js:433:35)
    at ~/bevis-stub/node_modules/enb/lib/server/server-middleware.js:65:33
    at Array.0 (~/bevis-stub/node_modules/vow/lib/vow.js:194:56)
    at callFns (~/bevis-stub/node_modules/vow/lib/vow.js:452:35)
```

----

Забегая вперёд, ответим на вопрос тех из вас, кто внимательно-превнимательно прочитал статьи и готов поймать нас на
лжи :) Мы говорили, что разработчику нужно думать только о директориях `blocks` и `pages`. Почему же в логах
фигурирует странный путь `build/test`, а не `pages/test`?

Потому что папка 'pages' - это директория для живых людей, а `build` - директория для робота сборщика и для
http-сервера. Сборщик складывает туда собранные файлы, а http-сервер читает их там и пересылает пользователю в
браузера. Пока думать о папке `build` не надо, наше дело - папка `pages`. О собираемых файлах
 мы будем думать, когда захотим кастомизировать сборку. Тогда же разберемся с консольным выводом — что сообщает
 каждая строчка. Сейчас не надо :)

----

Всё. Теперь создаём страницу `test`. В консоли прервём приложение сочетанием клавиш `Ctrl-C` или `Cmd-C` (если у вас
Mac OS). И воспользуемся инструментом для создания новой страницы. Это небольшой bash-скрипт, который задаст вам один
вопрос, а после создаст саму страницу:
```
make page
# Введите имя страницы: test
```

На файловой системе появится файл `pages/test-page/test-page.page.js` (рядом с ним ещё какой-то файл с расширением
`yaml`, он вам понадобится, но не сейчас, не обращайте внимания).

Откройте файл  `pages/test-page/test-page.page.js` в редакторе:

```javascript
module.exports = function (pages) {
    pages.declare('test-page', function (params) {
        var options = params.options;
        return {
            block: 'page',
            title: 'test page',
            styles: [
                {url: options.assetsPath + '.css'}
            ],
            scripts: [
                {url: options.assetsPath + '.js'}
            ],
            body: [
                // здесь ваши блоки
            ]
        };
    });
}
```

Вот такая страница.

Опять запустим проект
```
make
```

Обратите внимание на сообщения в консоли, они изменились. Теперь сборщик сообщает о том,
что он собирает не только `build/index/*`, но ещё и `build/test/*`

```
19:40:18.739 - build started
19:40:18.954 - [rebuild] [build/index/index.levels] levels
19:40:18.954 - [rebuild] [build/test/test.levels] levels
19:40:18.954 - [rebuild] [build/index/index.bemdecl.js] file-provider
19:40:18.955 - [isValid] [build/index/index.deps.js]
19:40:18.956 - [rebuild] [build/index/index.deps.js] deps-with-modules
19:40:18.956 - [rebuild] [build/index/index.files] files
19:40:18.956 - [rebuild] [build/index/index.dirs] files
19:40:18.957 - [isValid] [build/index/index.js] js
19:40:18.957 - [isValid] [build/index/index.css] css-stylus-with-autoprefixer
19:40:18.957 - [isValid] [build/index/_index.js] file-copy
19:40:18.957 - [isValid] [build/index/_index.css] file-copy
19:40:18.958 - [rebuild] [build/test/test.bemdecl.js] file-provider
19:40:18.958 - [isValid] [build/test/test.deps.js]
19:40:18.959 - [rebuild] [build/test/test.deps.js] deps-with-modules
19:40:18.959 - [rebuild] [build/test/test.files] files
19:40:18.959 - [rebuild] [build/test/test.dirs] files
19:40:18.959 - [isValid] [build/test/test.js] js
19:40:18.959 - [isValid] [build/test/test.css] css-stylus-with-autoprefixer
19:40:18.959 - [isValid] [build/test/_test.js] file-copy
19:40:18.959 - [isValid] [build/test/_test.css] file-copy
19:40:18.960 - build finished - 221ms

DEBUG: Running node-supervisor with
DEBUG:   program 'server/boot.js'
DEBUG:   --watch 'server,configs'
DEBUG:   --ignore 'undefined'
DEBUG:   --extensions 'node|js'
DEBUG:   --exec 'node'

DEBUG: Starting child process with 'node server/boot.js'
DEBUG: Watching directory '~/bevis-stub/server' for changes.
DEBUG: Watching directory '~/bevis-stub/configs' for changes.
19:40:19 - info: worker 23707 started
19:40:19 - info: app started on 8080
19:40:19 - info: worker 23708 started
19:40:19 - info: app started on 8080
```

Смотрим в браузере запрос к `localhost:8080/test/`

Сообщения об ошибке нет, появился серый аккуратный фоновый рисунок, а если заглянуть в html-код страницы,
увидим там такое:

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>test page</title>
    <link rel="stylesheet" href="/build/test/_test.css">
</head>
<body class="page _init" data-block="page">

    <script src="/build/test/_test.js" type="text/javascript"></script>
</body>
</html>
```

Это успех, у нас есть новая страница.

## Откуда взялся весь этот HTML?

Ещё раз посмотрим на декларацию файла `pages/test-page/test-page.page.js`.

```javascript
module.exports = function (pages) {
    pages.declare('index-page', function (params) {
        var options = params.options;
        return {
            block: 'page',
            title: 'index page',
            styles: [
                {url: options.assetsPath + '.css'}
            ],
            scripts: [
                {url: options.assetsPath + '.js'}
            ],
            body: [
                // здесь ваши блоки
            ]
        };
    });
}
```

В статьях мы обращали ваше внимание, что метод `pages.declare()` возвращает бивис-блоки:

```javascript
return [
    { block: 'header' },
    { block: 'authorization' }
];

```

и утверждали, что этого кода достаточно, чтобы Бивис сгенерил такой `html`:

```html
<div class="header">
    <a class="header__logo" href="/">
        <img src="logo.png">
    </a>
    <h1 class="header__title">Демо-страница</h1>
    <h2 class="header__slogan">Слоган</h2>
    <a class="header__rss" href="/?rss">
        <img src="rss.png">
    </a>
</div>
<form class="authorization" action="/?task=login">
    <label class="authorization__label">Логин</label>
    <input class="authorization__login" value="">
    <label class="authorization__label">Пароль</label>
    <input class="authorization__password"
        value="" type="password">
    <button class="authorization__submit">
        Войти
    </button>
</form>
```

Мы не обманули, так и есть. Но если вы вдумчивый и въедливый, то уже задались вопросом,
как именно генерятся обязательные для любой веб-страницы теги `html`, `head`, `body`.

Мы нарочно опустили этот момент, потому что он не предсталяет отдельной ценности и сложности. Потому что структура
страницы генерится отдельным блоком `page`, который поставляется сразу с бивисом,
и автоматически генерится при  создании заготовки новой страницы.

Когда мы хотим наполнить страницу какими-то блоками, мы делаем это через декларацию этих блоков в
блоке `page`, например так:

```javascript
module.exports = function (pages) {
    pages.declare('test-page', function (params) {
        var options = params.options;
        return {
            block: 'page',
            title: 'test page',
            styles: [
                {url: options.assetsPath + '.css'}
            ],
            scripts: [
                {url: options.assetsPath + '.js'}
            ],
            body: [
                { block: 'ВАШ БЛОК НОМЕР 1' },
                { block: 'ВАШ БЛОК НОМЕР 2' }
            ]
        };
    });
}
```

Блок `page`, который здесь объявлен, есть в папке `/blocks`. Он формирует минимальную `html` структуру
любой страницы и имеет несколько параметров, назначение которых легко понять из имён.

В `title` мы передаём заголовок страницы, который окажется в теге `<title>` секции `<head>`.
Параметры `styles` и `scripts` принимают массив стилей и скриптов соответственно. Они генерят теги `<link>` или
`<styles>` и `<script>`.

Параметр `body` тоже ожидает на вход массив. Даже если в теле web-страницы вы хотите видеть один единственный блок,
всё равно параметр `body` ожидает от вас массив. Это очень простой параметр. Можно относиться к нему,
как контейнеру `<body>` из обычного `html`-файла.

Детально узнать об API блока `page` можно из `jsdoc` в файле `blocks/page/page.bt.js`

Страница готова к наполнению блоками. Приготовим какой-нибудь блок?

Нам [сюда](new-block.md).

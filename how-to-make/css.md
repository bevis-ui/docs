в разработке

----

Первые два шага мы уже сделали — [создали новую страницу](new-page.md), [создали новый блок](new-block.md).

У нас уже есть страница `/pages/test-page/test-page.page.js`, а в ней единственный блок `input`:

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
                {
                    block: 'input',
                    value: 'Привет, Бивис!'
                }
            ]
        };
    });
}
```

Ещё по адресу `/blocks/input/` есть `bt`-шаблоны для `input`, стилевой файл и файл с `JS`-поведением этого блока.
Откроем стилевой файл `/blocks/input/input.styl` и начнём :)

Если вы выполнили шаги в [предыдущем руководстве](new-block.md), то файл у вас выглядит совершенно бесполезно.
У меня там написано вот что:

```css
.input {
    border: 1px solid red;
}
```

Откроем в браузере `localhost:8080/test/`. Блок `input` выглядит непрезентабельно - голое текстовое поле
с красной обводкой. Именно это мы и будем это исправлять.

## Дизайнер дал картинку

Вот реальный эскиз из Яндекса.

<img src="http://bevis-ui.github.io/docs/images/input-design.png" width="557" height="539" />

Дизайн без особых затей:

* текстовое поле плоское, белый фон, серая рамка
* жёлтое свечение вокруг поля, когда поле получает фокус
* текст-приглашение, если поле пустое
* крестик для очистки поля

Ещё дизайнер предусмотрел разные размеры блока. И, конечно, предусмотрел неактивное состояние,
когда поле недоступно  для действий пользователя.

Глядя на эскиз, становится очевидно, что одним DOM-узлом мы не обойдёмся, придётся верстать дополнительный
элемент `крестик`, потому что такого нативного браузерного контрола, к сожалению, не существует. Значит блок `input`
придётся усложнить. Сейчас он выглядит так:

```html
<input class="input" type="text"/>
```

А мы сделаем его таким:

```html
<div class="input">
    <input class="input__control" type="text" placeholder="" value="" name=""/>
    <div class="input__close"></div>
</div>
```

Текстовое поле, как и элемент "крестик" станут детьми блока, его элементами. И как мы видим,
у текстового поля появится атрибут `placeholder`, в котором и будет храниться тот самый
текст-приглашение.

Мы не хотим хардкодить текст-приглашение, потому что блок `input` получится универсальным блоком,
и нам захочется использовать его в разных формах. По той же причине мы не будем хардкодить `name` текстового поля.
Вместо этого мы расширим API блока — разрешим при декларации блока указывать значения для этих атрибутов. Для этого  в
`/pages/test-page/test-page.page.js` в вызове блока добавим два новых поля - `name` и `placeholder`:

```javascript
{
    block: 'input',
    value: 'Привет, Бивис!',
    name: 'loginField',
    placeholder: 'на сайте'
}
```

И теперь поддержим все изменения в шаблонах.

## Усложним HTML-структуру

Откроем файл `/blocks/input/input.bt.js`, сейчас он выглядит вот так:

```javascript
module.exports = function (bt) {

    bt.match('input', function (ctx) {
        ctx.enableAutoInit();
        ctx.setTag('input');

        var currentValue = ctx.getParam('value');
        ctx.setAttr('value', currentValue);
    });

};
```

Для начала перестанем выливать тег `input`. Пусть наш блок будет блочным DOM-узлом,
а уже внутри него пусть находятся два элемента - текстовое поле и крестик:

```javascript
module.exports = function (bt) {

    bt.match('input', function (ctx) {
        ctx.enableAutoInit();

        var value = ctx.getParam('value');
        var name = ctx.getParam('name');
        var placeholder = ctx.getParam('placeholder');

        ctx.setContent([
            {
                elem: 'control',
                inputValue: value,
                inputName: name,
                placeholder: placeholder
            },
            {
                elem: 'close'
            }
        ]);
    });

};
```

Мы выкинули метод `ctx.setTag('input')`, потому что обертка должна быть тегом `div`,
а такой тег генерится шаблонизатором по умолчанию даже если мы этого не указали явно.

Мы создали локальные переменные `value`, `name`, `placeholder` и сохранили в них значения параметров из контекста с
помощью метода `ctx.getParam()`. Из какого такого контекста? В контексте чего работает шаблон блока? Шаблон блока
работает в контексте данных, которые описаны в `test-page.page.js`, то есть фактически в контексте этого
фрагмента страницы:

 ```javascript
{
    block: 'input',
    value: 'Привет, Бивис!',
    name: 'loginField',
    placeholder: 'на сайте'
}
```

Затем мы вызвали метод `ctx.setContent()`, в который передали массив элементов. Мы решили,
что внутри блока будут жить два элемента - те самые `input__control` и `input__close`. Элементы описываются так же,
как блок, с той лишь разницей, что вместо слова `block` мы зовём элемент словом `elem`.

Если сейчас обновить страницу в браузере, текстовое поле со страницы исчезнет, но заглянув в HTML-код страницы,
мы увидим почти то, что нужно:

```html
<div class="input" data-block="input">
    <div class="input__control"></div>
    <div class="input__close"></div>
</div>
```

Осталось сгенерить правильный тег для `input__control` и передавать в него значение для атрибутов
`value`, `name` и `placeholder`. Для этого добавим новый шаблон:

```javascript
module.exports = function (bt) {

    bt.match('input', function (ctx) {
        ctx.enableAutoInit();

        var value = ctx.getParam('value');
        var name = ctx.getParam('name');
        var placeholder = ctx.getParam('placeholder');

        ctx.setContent([
            {
                elem: 'control',
                inputValue: value,
                inputName: name,
                placeholder: placeholder
            },
            {
                elem: 'close'
            }
        ]);
    });

    bt.match('input__control', function (ctx) {
        ctx.setTag('input');

        var currentValue = ctx.getParam('inputValue');
        var currentName = ctx.getParam('inputName');
        var currentPlaceholder = ctx.getParam('placeholder');

        ctx.setAttr('value', currentValue);
        ctx.setAttr('name', currentName);
        ctx.setAttr('placeholder', currentPlaceholder);
    });

};
```

В шаблоне для `input__control` контекст меняется. Шаблон этого элемента работает в контексте данных,
которые пришли к нему из предыдущего шаблона, то есть в контексте вот этого `btjson`:

```javascript
{
    elem: 'control',
    inputValue: ctx.getParam('value'),
    inputName: ctx.getParam('name'),
    placeholder: ctx.getParam('placeholder')
}
```

Именно поэтому в шаблоне элемента `control` мы читаем ожидаем параметры под другими именами (я специально назвал их не
теми же переменными, что в родительском шаблоне, чтобы вы ухватили идею):

```javascript
var currentValue = ctx.getParam('inputValue');
var currentName = ctx.getParam('inputName');
var currentPlaceholder = ctx.getParam('placeholder');
```

Смотрим в браузере. Да, теперь всё отлично, структура приходит.

```html
<div class="input _init" data-block="input">
    <input class="input__control" value="Привет, Бивис" name="loginField" placeholder="на сайте">
    <div class="input__close"></div>
</div>
```

Наконец-то займёмся стилями.

## Стили для блока и его элементов

## Стили состояний

## Что мы пишем для IE

## Нужны разные отображения?

_Переписать для разных размеров вместо абстрактных вью!_

Предположим, ваш дизайнер для разных форм сайта нарисовал разные текстовые поля.
И предположим, у вас уже есть блок `input`, который мы сверстали в [соседнем документе](new-block.md). Будем решать
задачу на основе этого блока.

Есть смысл сделать два  отдельных представления этого блока, два `view`. У них будут разные стили,
но одинаковое поведение.

Придумаем имена для представлений: `normal` и `active` и добавим поддержку `view` в шаблоны.

Одно из отображений можно сделать умолчательным, например `normal`, потому что именно оно используется
 на всех страницах сайта, а `active` только на одной странице. Умолчательным (по дефолту,
 если говорить на привычном жаргон) означает, что в декларации блока не нужно указывать `view: 'normal'`,
 это вью будет добавляться автоматически.

Правим шаблоны:

* добавляем вызов метода `bt.setDefaultView('input', 'normal')`
* и к имени блока добавляем маску `*`

Было:

```javascript
module.exports = function (bt) {
    bt.match('input', function (ctx) {
        // ...
    });
};
```

Стало:

```javascript
module.exports = function (bt) {
    bt.setDefaultView('input', 'normal'); // Добавил вызов метода
    bt.match('input*', function (ctx) { // Добавил *-маску в имя блока
        // ...
    });
};
```

Теперь шаблон отработает для блока с любым `view`, то есть для любого из нижеописанных вызовов:
```javascript
{
    block: 'input',
    view: 'normal',
    value: 'Hello'
}
// Сгенерит <div class="input_normal"></div>

{
    block: 'input',
    view: 'active',
    value: 'Hello'
}
// Сгенерит <div class="input_active"></div>

{
    block: 'input',
    value: 'Hello'
}
// Сгенерит <div class="input_normal"></div>

// Это при том, что view мы не указывали.
// Дефолтное значение normal подставится автоматически благодаря методу bt.setDefaultView('input', 'normal')
```

Проверям в браузере, так ли это? Нет, не заработало.

Потому что нет зависимостей.

* зависимости для вью

Проверяем: работает. В html выливаются правильные классы.

Теперь расширяем стили

* два файла styl
* селекторы с вью
* общшие стили выносим в миксины скинов.

Проверяем - работает.



----

### Предварительный план:
1. есть тестовая страница (ссылка) и блок инпут (ссылка) на ней. Если нет - создайте.
2. Стилей на блоке ещё нет. Давайте напишет. Пусть инпут выглядит как на картинке (скрин из яндекс-инпута с крестиком)
3. Изменим структуру: bt: обвязка>инпут+крестик
4. Напишем стили для блока и двух элементов
5. Состояния: _disabled _focused
6. Полпрозрачная рамка (бордер+bg-clip) - только ради хвастовства и показать как хачим ИЕ
7. Как хачим для ИЕ (в ie8 показываем if-else)
8. Нужно два разных вью (найти дизайн инпута, только чуть-чуть другой)? Как?

### В тексте подчеркнуть тезисы:
* Чтобы определить html-структуру блока и задать ему оформление, мы используем параметр `view`.
* Параметр `view` можно задавать только в `btjson`. Мы считаем, что блок не может менять свои представление и html-структуру в браузере. Если это происходит, мы считаем, что блок становится чем-то иным, нежели он есть — становится другим блоком.
* `state` — состояния блока. Использовать их без конкретного view бессмысленно, потому что состояния существуют только в рамках конкретного варианта оформления. Состояния могут меняться прямо на странице в браузере.
* Миксины расположены в модификаторе `skin`, и подключаются в `view` с помощью `deps`-файлов.

----


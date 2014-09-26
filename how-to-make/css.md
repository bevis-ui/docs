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
    <div class="input__clear"></div>
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
                    value: 'Привет, Бивис!',
                    name: 'loginField',
                    placeholder: 'на сайте'
                }
            ]
        };
    });
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

Пусть блок `input` превратится в небольшой фрагмент DOM-дерева с двумя дочерними элементами - собственно текстовое поле
 и крестик для очистки текстового поля:

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
                elem: 'clear'
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

То есть, когда мы в шаблоне сказали, к примеру, `var placeholder = ctx.getParam('placeholder')`,
то в локальной переменной `placeholder` окажется строка "на сайте", которую мы указали в декларации страницы — мы
получили значение параметра из контекста.

Затем мы вызвали метод `ctx.setContent()`, в который передали массив элементов. Мы решили,
что внутри блока будут жить два элемента - те самые `input__control` и `input__clear`. Элементы описываются так же,
как блок, с той лишь разницей, что вместо слова `block` мы зовём элемент словом `elem`.

Если сейчас обновить страницу в браузере, текстовое поле со страницы исчезнет, но заглянув в HTML-код страницы,
мы увидим почти то, что нужно:

```html
<div class="input" data-block="input">
    <div class="input__control"></div>
    <div class="input__clear"></div>
</div>
```

Осталось сгенерить правильный тег для `input__control` и передавать в него значение для атрибутов
`value`, `name` и `placeholder`. Для этого добавим новый шаблон (смотрите ниже,
там появился шаблон для `input__control`):

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
                elem: 'clear'
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
ctx.setContent([
{
    elem: 'control',
    inputValue: value,
    inputName: name,
    placeholder: placeholder
}
```

Именно поэтому в шаблоне элемента `control` мы ожидаем параметры под другими именами. Я специально назвал их иначе,
нежели в родительском шаблоне, чтобы вы ухватили идею.

Ещё раз о контексте, потому что это важный момент.

```javascript
// В шаблоне для `input` я получил параметры из контекста `test-page.page.js`
var value = ctx.getParam('value');
var name = ctx.getParam('name');
var placeholder = ctx.getParam('placeholder');

// а потом прокинул их значения в элемент `control`
ctx.setContent([
{
    elem: 'control',
    inputValue: value,
    inputName: name,
    placeholder: placeholder
}
```

```javascript
// В шаблоне дочернего `input__control` я получил ровно эти же значения,
// но уже из параметров самого элемента `control`
var currentValue = ctx.getParam('inputValue');
var currentName = ctx.getParam('inputName');
var currentPlaceholder = ctx.getParam('placeholder');

// и создал атрибуты для html-тега:
ctx.setAttr('value', currentValue);
ctx.setAttr('name', currentName);
ctx.setAttr('placeholder', currentPlaceholder);
```

Теперь смотрим в браузере. Да, всё отлично, структура приходит.

```html
<div class="input _init" data-block="input">
    <input class="input__control" value="Привет, Бивис" name="loginField" placeholder="на сайте">
    <div class="input__clear"></div>
</div>
```

Наконец-то займёмся стилями.

## Стили для блока и его элементов

Возвращаемся в `/blocks/input/input.styl` и начнём :)

Если вы выполнили шаги в [предыдущем руководстве](new-block.md), то файл у вас выглядит совершенно бесполезно.
У меня там написано вот что:

```css
.input {
    border: 1px solid red;
}
```

Напишем стили, реализующие задумку дизайнера.

```css
.input {
    $height = 26px;

    position: relative;
    display: inline-block;
    box-sizing: border-box;

    background-color: #fff;
    border: 1px solid rgba(0, 0, 0, 0.2);

    // чтобы под бордером не было белого фона
    background-clip: padding-box;

    vertical-align: baseline;
    line-height:  normal;
    cursor: text;

    transition:box-shadow .05s ease-out; /* анимация желтой фокусной рамки */

    /* <input> */
    &__control{
        font-family: Arial, Helvetica, sans-serif;
        font-size: 13px;

        position: relative;
        display: inline-block;
        box-sizing: border-box;
        margin: 0;
        padding: 0 .6em;
        width: 100%;
        height: $height;
        // Чтобы не заползал под крестик
        padding-right: $height;

        border: none;
        vertical-align: baseline;

        &:focus{
          outline: none;
        }

        /*
            @see http://alexbaumgertner.ya.ru/replies.xml?item_no=2112
            @see http://www.cssnewbie.com/input-button-line-height-bug/
        */
        &::-moz-focus-inner {
            padding: 0;
            border: none;
        }


        /* прячем стандартный крестик IE10 */
        &::-ms-clear {
            display: none;
        }
    }

    &._focused {
        outline: 0;
        box-shadow: 0 0 10px #fc0;

        if (ie == 8) {
            border-color: #d1bb66;
        } else {
            border-color: rgba(178, 142, 0, .6);
        }
    }

    &._disabled{
        border-color: transparent !important;
        cursor: default;

        if (ie == 8) {
            background-color: #ebebeb !important;
        } else {
            background-color: rgba(0, 0, 0, 0.08) !important;
        }
    }

    &._disabled &__control{
        user-select: none;
        background-color: inherit;
    }

    &__clear {
        position: absolute;
        top: 0;
        right: 0;
        z-index: 2;

        width: $height;
        height: $height;
        vertical-align: middle;

        background: url('images/clear-10.png') 50% 50% no-repeat;
        cursor: pointer;
        opacity: (ie != 8) ? 0.3 : 1;

        &:hover {
            background-image: url('images/clear-10-hover.png');
            opacity: 1;
        }

        &._visible {
            display: block;
        }
    }
}
```

Обычный `CSS`-код с базовыми конструкциями препроцессора: переменные (`$height`), условные операторы (`if`) и ссылка на 
родительский селектор `&`. Возможно, вы захотите использовать циклы, перебирающие методы и другие богатсва `Stylus`. Мы 
же сознательно не используем эти фичи, чтобы код оставался максимально похожим на `Plain CSS` — простым и наглядным. 

Скопировали? Теперь обновите страницу в браузере. Не хватает картинки для крестика. Не хватает жёлтой рамки вокруг блока
 при установке фокуса. А в остальном - похоже на то, что хочет дизайнер.

Стили незамысловатые, селекторы понятные, поэтому о них всего по паре слов.

```css
    background-color: #fff;
    border: 1px solid rgba(0, 0, 0, 0.2);

    // чтобы под бордером не было белого фона
    background-clip: padding-box;
```

Дизайнер придумал для блока темно-серый полупрозрачный бордер. Страница у нас имеет светло-серый фон, бордеру задаём 
`rgba(0, 0, 0, 0.2)` и ждём, что скзвозь бордер мы увидим фон страницы. Но почему-то сквозь бордер мы видим белый 
фоновый цвет. Почему?                                                                                    

Бордер по всем спецификациям, откладывается _вне_ блока. Но фоновый цвет, заданный блоку, почему-то оказывается и 
под бордером тоже. Такое поведение фона цвета кажется необычным с точки зрения человеческой логики, даже если оно по 
спецификации. Вот для чего нужен `background-clip: padding-box` — этим мы заставляем фоновый цвет ограничиваться зоной 
внутренних отступов. Это такая хитрость, вдруг вы раньше не сталкивались, а теперь будете знать. 
  
Нативный браузерный `<input>` в нашем блоке представлен элементом с классом `.input__control`. В `Stylus` мы описываем его
 в селекторе `&__control {}`, где `&` - ссылка на родительский селектор `.input{}`. Нам нужно лишить нативный браузерный 
 контрол `&__control {}` всякого нативного оформления, а родительский блок `.input{}` наоборот оформить согласно 
 дизайну. 
 
Про фокус. Так как браузерный `<input>` оказался внутри нашего кастомного блока, стал безликим и словно растворился в 
дизайне, мы больше не можем разрешать браузеру показывать дефолтный фокус на `<input>`. Отменяем:

```css
    /* <input> */
    &__control{

        &:focus{
          outline: none;
        }

        /*
            @see http://alexbaumgertner.ya.ru/replies.xml?item_no=2112
            @see http://www.cssnewbie.com/input-button-line-height-bug/
        */
        &::-moz-focus-inner {
            padding: 0;
            border: none;
        }
    }
```

Но фокус показывать мы должны, поэтому эмулируем его на самом блоке с помощью специального `состояния блока`. Блок 
оказался в фокусе. 
 
## Стили состояний

Состояние, оно же `state`, выражается специальным `CSS`-классом, который _немного_ (это важное уточнение) изменяет 
внешний вид блока. Чаще всего состояние меняется из `javascript`. 

Например, на блок нажали — у блока появилось состояние `pressed`. Блок неактивный — у него сразу выставлено состояние 
`disabled`

Здесь описано состояние, когда блок `input` оказался в фокусе.

```css
    &._focused {
        outline: 0;
        box-shadow: 0 0 10px #fc0;

        if (ie == 8) {
            border-color: #d1bb66;
        } else {
            border-color: rgba(178, 142, 0, .6);
        }
    }
```

Жёлтое свечение вокруг блока. Однопиксельный бордер, почти невидимый. В IE8 бордер сплошной без прозрачности.

Кстати, для IE ничего особенного не пишем.
            
Простые условия:

```css
        if (ie == 8) {
            border-color: #d1bb66;
        } else {
            border-color: rgba(178, 142, 0, .6);
        }
```

Или тернарные:

```css
        opacity: (ie != 8) ? 0.3 : 1;
```

Ничего сложнее, кажется, и не нужно. 

Классы состояний мы начинаем с символа подчёркивания. 

В программировании символ подчёркивания в начале имени переменной иллюстрирует, что эта перемнная — приватная. 
Мы в `BEViS` этим же символом иллюстрируем, что состояние — тоже приватно. Сам символ ни на что не влияет, спокойно 
можно и без него жить. Но нам нравится, мы используем. 

Состояние само по себе ничего не делает. Оно лишь _добавляет_ или _переопределяет_ стили, чтобы блок сделать чуть-чуть 
 иным — раскрасить чуть иначе, в соответствии с текущим состоянием.

Классам состояний мы даём имена, как если бы они были образованы для формирования страдательного залога английского 
языка. Непонятно? Я бы и сам не понял, прочитав такое. А можно проще:

"Block is focused" — отбрасываем "Block is" — что осталось, то и пишем.
 
"Block is disabled", "Block is visible", "Block is active". Нет догмы. Мы называем так, вам может оказаться удобно 
иначе, не проблема.

Блок может одновременно находиться в разных состояниях: `_focused` и `_pressed`, `_visible` и `_disabled`. 

Так как состояния доопределяют или переопределяют стили блока, то сочетания разных состояний грозит возможными 
конфликтами между селекторами. Или скажем иначе: "Не грозит, а грозило бы, если в BEViS не существовало жёсткого 
правила — на одной html-ноде может быть только один блок". А если так, тогда всё многообразие состояний блока существует 
 в пределах одного стилевого файла. И мы способны все сочетания состояний продумыват заранее. 

Что будет, когда блок нажат и стоит в фокусе? Подумаем и напишем:

```css
    &._focused._pressed {
        /* здесь стили */
    }
```

А когда задизейблен и не скрыт? Напишем:

```css
    &._disabled._visible {
        /* здесь стили */
    }
```


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


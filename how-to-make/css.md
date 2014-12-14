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

    /* чтобы под бордером не было белого фона */
    background-clip: padding-box;

    vertical-align: baseline;
    line-height:  normal;
    cursor: text;

    /* анимация желтой фокусной рамки */
    transition:box-shadow .05s ease-out;

    /* <input> */
    &__control {
        font-family: Arial, Helvetica, sans-serif;
        font-size: 13px;

        position: relative;
        display: inline-block;
        box-sizing: border-box;
        margin: 0;
        padding: 0 .6em;
        width: 100%;
        height: $height;
        /* Чтобы не заползал под крестик */
        padding-right: $height;

        border: none;
        vertical-align: baseline;

        &:focus {
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

    &._disabled {
        border-color: transparent !important;
        cursor: default;

        if (ie == 8) {
            background-color: #ebebeb !important;
        } else {
            background-color: rgba(0, 0, 0, 0.08) !important;
        }
    }

    &._disabled &__control {
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

        background: url('images/clear-14.png') 50% 50% no-repeat;
        cursor: pointer;
        opacity: (ie != 8) ? 0.3 : 1;

        &:hover {
            background-image: url('images/clear-14-hover.png');
            opacity: 1;
        }

        &._visible {
            display: block;
        }
    }
}
```

Обычный `CSS`-код с базовыми конструкциями препроцессора: переменные (`$height`), условные операторы (`if`) и ссылка на
родительский селектор `&`. Возможно, вы захотите использовать циклы, перебирающие методы и другие богатства `Stylus`. Мы
 сознательно не используем все возможности препроцессора, чтобы код оставался максимально похожим на `Plain CSS` — простым и наглядным.

Скопировали? Теперь обновите страницу в браузере. Не хватает картинки для крестика. Не хватает жёлтой рамки вокруг блока
 при установке фокуса. А в остальном похоже на то, что хочет дизайнер.

Стили незамысловатые, селекторы понятные, поэтому о них всего по паре слов.

```css
    background-color: #fff;
    border: 1px solid rgba(0, 0, 0, 0.2);

    /* чтобы под бордером не было белого фона */
    background-clip: padding-box;
```

Дизайнер придумал для блока темно-серый полупрозрачный бордер. Страница у нас имеет светло-серый фон, бордеру задаём
`rgba(0, 0, 0, 0.2)` и ждём, что скзвозь бордер мы увидим фон страницы. Но почему-то сквозь бордер мы видим белый
фоновый цвет блока. Почему?

Бордер, по всем спецификациям, откладывается _вне_ блока. Но фоновый цвет, заданный блоку, почему-то оказывается и
под бордером тоже. Такое поведение фона цвета кажется необычным с точки зрения человеческой логики, даже если оно по
спецификации. Вот для чего нужен `background-clip: padding-box` — этим мы заставляем фоновый цвет ограничиваться зоной
внутренних отступов. Это такая хитрость, вдруг вы раньше не сталкивались, а теперь будете знать.

Нативный браузерный `<input>` в нашем блоке представлен элементом с классом `.input__control`. В `Stylus` мы описываем его
 в селекторе `&__control {}`, где `&` - ссылка на родительский селектор `.input{}`. Нам нужно лишить нативный браузерный
 `<input>` всякого нативного оформления, а родительский блок `.input{}` наоборот оформить согласно
 дизайну.

Про фокус. Так как браузерный `<input>` оказался внутри нашего кастомного блока, стал безликим и словно растворился в
дизайне, мы больше не можем разрешать браузеру показывать дефолтный фокус на `<input>`. Отменяем:

```css
    /* <input> */
    &__control {

        &:focus {
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

В программировании символ подчёркивания в начале имени переменной иллюстрирует, что эта переменная — приватная.
Мы в `BEViS` этим же символом иллюстрируем, что состояние — тоже приватно. Сам символ ни на что не влияет, спокойно
можно и без него жить. Но нам нравится, мы используем.

Состояние само по себе ничего не делает. Оно лишь _добавляет_ или _переопределяет_ стили, чтобы блок сделать чуть-чуть
 иным — раскрасить чуть иначе, в соответствии с текущим состоянием.

Классам состояний мы даём имена, как если бы они были образованы для формирования страдательного залога английского
языка. Непонятно? Я бы и сам не понял, прочитав такое. А можно проще:

"Block is focused" — отбрасываем "Block is" — что осталось, то и пишем.

"Block is disabled", "Block is visible", "Block is active". Мы называем так. Вы можете называть иначе. Главное, чтобы
было удобно. Догмы нет.

Блок может находиться в разных состояниях одновременно: `_focused` и `_pressed`, `_visible` и `_disabled`. Комбинации
могут быть самые разные, хоть все сразу.

Так как состояния доопределяют или переопределяют стили блока, то сочетания разных состояний грозит возможными
конфликтами между селекторами. Или скажем иначе: "Не грозит, а грозило бы, если в BEViS не существовало жёсткого
правила — на одной html-ноде может быть только один блок". А если так, тогда всё многообразие состояний блока существует
 в пределах одного стилевого файла. И мы способны все сочетания состояний продумывать заранее.

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

Стили готовы, но увидеть их в браузере польностью мы не можем, потому что ещё не написан скрипт, меняющий состояния.
 Скрипт мы будем писать позже и в [отдельном документе](yblock.md), а здесь только про стили.

## Нужны разные отображения?

Дизайнер нарисовал текстовые поля в разных размерах — у них совершенно одинаковые стили, кроме размеров, каких-то
отступов, величины гарнитуры шрифта. Нарисовал целых 4 размера и назвал их, как маркируют бирки на одежде - S, M, L, XL.

В таких случаях верстальщики создают базовый класс `.input`, соответствующий дефолтному размеру, и 3 дополнительных,
модифицирующих базовый класс, определяющих другие размеры и отступы. Мы называли их `.small`, `.large`, `.xlarge` и
всё это выглядело бы примерно так:

```css
.input {
    /* все базовые стили */
}

.input.small {
    height: 16px;
    font-size: 11px;
    padding: 2px;
}

.input.large {
    height: 22px;
    font-size: 18px;
    padding: 6px;
}

.input.xlarge {
    /* и здесь уточнения для XL */
}
```

Если внимательно вглядеться в эти составные селекторы, легко заметить, что они читаются как человеческие имена: "Инпут,
Маленький Инпут, Большой Инпут, Очень Большой Инпут". Почему же эти имена выражены через два `CSS`-класса?

В Бивисе мы создаём четыре отдельных класса - по одному для каждого размера:

```css
.input_normal {
    /* все базовые стили + стили для размера M */
}

.input_small {
    /* все базовые стили + стили для размера S */
}

.input_large {
    /* все базовые стили + стили для размера L */
}

.input_xlarge {
    /* все базовые стили + стили для размера XL */
}
```

Каждый такой класс будет в себе содержать все базовые стили (которые раньше хранились в классе `.input`) и нужные
размеры. Если мы так сделаем, код станет простым и очевидным — для каждого размера свой класс, который реализует только
нужное отображение. Представление.

Эти четыре селектора выше - это `представления` или `отображения` или `view` блока. Инпут может иметь нормальное
представление (`normal`), маленькое (`small`), большое (`large`) или любое другое, которое мы ему создадим.

Сразу решим ваши сомнения. Дублируется ли код базовых стилей блока в каждом вью? Нет:

```css
input_mixin() {
    /* все базовые стили */
}

.input_normal {
    input_mixin();
    /* здесь только стили для размера M */
}

.input_small {
    input_mixin();
    /* здесь только стили для размера S */
}

.input_large {
    input_mixin();
    /* здесь только стили для размера L */
}

.input_xlarge {
    input_mixin();
    /* здесь только стили для размера XL */
}
```

Миксины - это четвертая и последняя возможность, которую мы используем в `Stylus`. Миксин - молчаливая функция, которая
не попадает в конечный `CSS` сама по себе. Но если её в каком-то селекторе позвать, она вернёт вместо себя стили,
описанные внутри.

Это мы и делаем, чтобы обеспечить каждое представление инпута базовыми стилями. Именно внутрь `input_mixin()` мы
перенесём все стили, которые сейчас описаны в `input.styl`. Откройте его на редактирование и замените первую строку
файла, чтобы вместо `.input {` получилось `input_mixin() {`. Обратите внимание - нужно удалить точку (это не селектор,
а функция) и дописать круглые скобки (что лишний раз показывает, что это всё-таки функция, а не селектор).

Если сейчас обновить страницу в браузере, блок потеряет все стили. Ну вот, миксин - не селектор, сам по себе не
применяется.

Допишем в конец файла селектор для Большого Инпута. У вас должно получиться так:

```css
input_mixin() {
    $height = 26px;

    /* все остальные базовые стили */
}

.input_large {
    input_mixin($height = 32px);

    &__control{
        font-size: 16px;

        padding: 0 1em;
    }
}
```

Обратили внимание, что в миксин мы передаём параметр? Поддержим этот параметр в самом миксине и удалим жёстко
зашитую переменную:

Было:
```css
input_mixin() {
    $height = 26px;
```


Сделаем так:
```css
input_mixin($height) {

```

Теперь мы принимаем значение высоты, как аргумент миксина.

На всякий случай сверим содержимое файла `input.styl`. У вас там сейчас написано следующее? Если нет, самое время
поправить, чтобы было именно так:

```css
input_mixin($height) {

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

    &._focused {
        outline: 0;
        box-shadow: 0 0 10px #fc0;

        if (ie == 8) {
            border-color: #d1bb66;
        } else {
            border-color: rgba(178, 142, 0, .6);
        }
    }

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

        background: url('images/clear-14.png') 50% 50% no-repeat;
        cursor: pointer;
        opacity: (ie != 8) ? 0.3 : 1;

        &:hover {
            background-image: url('images/clear-14-hover.png');
            opacity: 1;
        }

        &._visible {
            display: block;
        }
    }
}

.input_large {
    input_mixin($height = 32px);

    &__control{
        font-size: 16px;

        padding: 0 1em;
    }
}
```

Обновим страницу в браузере. Ничего не изменилось. Смотрим в html - видим, что блок выглядит так

```html
<div class="input _init" data-block="input">
    <input class="input__control" value="Привет, Бивис" name="loginField" placeholder="на сайте">
    <div class="input__clear"></div>
</div>
```

Уверен, вы поняли что пошло не так. Мы написали селектор для блока с вью — `input_large`, а в файле `test-page.page.js` мы декларируем
инпут без всякого вью. Поэтому стили не применяются.

Поправим декларацию блока. Добавим `view: 'large'`

```javascript
{
    block: 'input',
    view: 'large',
    value: 'Привет, Бивис',
    name: 'loginField',
    placeholder: 'на сайте'
}
```

Обновим страницу в браузере. Блок вообще исчез! Смотрим в файрбаге - видим только обертку блока:

```html
<div class="input_large" data-block="input"></div>
```

Если внутренности не генерятся, это значит, не отрабатывают `bt`-шаблоны.

И действительно. Шаблоны сейчас матчатся на `input`, а у нас уже не просто `input`, у нас уже `input_large`.

Решение простое - переписать match-и c `match('input'` на `match('input_large'`.

Переписали, смотрим в браузере - да! Теперь инпут стал большим.

Теперь мы можем в `input.styl` написать ещё один селектор для Маленького Инпута? И ещё один для Нормального Инпута? А
для Очень Большого, для Красного Инпута, для Серо-Буро-Малинового? Для любого можем и оно заработает?

Нет, потому что мы умудрились с вами захардкодить имя вью в шаблонах, когда переписали матчи на на `match('input_large'`
Эти шаблоны применятся _только_ для Большого Инпута. Мы можем в шаблонах использовать "маски".

К имени блока вместо конкретного вью добавьте маску `*`:

Было:
```javascript
module.exports = function (bt) {
    bt.match('input_large', function (ctx) {
        // ...
    });
};
```

Стало:
```javascript
module.exports = function (bt) {
    bt.match('input*', function (ctx) {
        // ...
    });
};
```

Вот теперь эти шаблоны будут отрабатывать для инпутов с любым `view`, то есть для любого из нижеописанных вызовов:

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
```

А ещё можно задавать какое-либо `view` как дефолтное. Чтобы даже если вы не задекларировали никакого `view`, оно
сгенерировалось бы само. Это нужно, когда на сайте большинство инпутов имеет размер `normal`. Тогда мы везде
декларируем блок без `view`:

```javascript
{
    block: 'input',
    value: 'Hello'
}
```

А чтобы построился такой html `<div class="input_normal"></div>`, мы в файле с шаблонами объявляем дефолтный `view` с
помощью метода `bt.setDefaultView('input', 'normal');`

Было:
```javascript
module.exports = function (bt) {

    bt.match('input*', function (ctx) {
// ...
```

Стало
```javascript
module.exports = function (bt) {
    bt.setDefaultView('input', 'normal');

    bt.match('input*', function (ctx) {
// ...
```

И вот теперь наш блок готов работать с любыми `view`. Как только нам понадобится новое представление, мы создаём новый
селектор для представления, звоём в него базовый миксин и доопределяем или переопределяем стили базового миксина.

И всё.

Сложное объяснение? Ээх, оно таким сложным выглядит только на бумаге. Если бы я объяснял вам это же голосом, вам бы оно
показалось простым. :(

## Файловая система для `view`

У нас получилось так, что все-все стили оказались в одном файле. Файл рискует стать большим. На наш взгляд это совсем
 неудобно. Так можно. Но нам неудобно. Мы стараемся разбивать стили на файлы согласно простому правилу: "Миксин - в
отдельный файл с именем миксина, вью - в отдельный файл с именем вью".

Работать с таким кодом легче, мы редактируем всегда только то, что нужно в данный момент,
не рискуя задеть код соседних вью или, не дай боже, миксина. Как это организовать?

Создайте файл `input_large.styl`. Это будет файл для вью "Большой Инпут". В него из файла `input.styl` перенесите
селектор для этого вью:

```css
.input_large {
    input_mixin($height = 32px);

    /* <input> */
    &__control{
        font-size: 16px;

        padding: 0 1em;
    }
}
```

А в файле `input.styl` таким образом останется только код миксина `input_mixin($height) {...}`.

Проверяем в браузере, стили исчезли. Почему? Сейчас научу, как понять. Откройте собранный `css`-файл. Он находится по
 адресу `/build/test/test.css`. Там увидите такое:

```css
/* /bevis/blocks/page/page.styl:begin */
.page {
  background: #eaeaea url(../../blocks/page/page.png);
}
/* /bevis/blocks/page/page.styl:end */
/* /bevis/blocks/input/input.styl:begin */
/* /bevis/blocks/input/input.styl:end */
```

Видно, что сборщик заглядывает в файл `input.styl`, но оттуда ничего не забирает, потому что там только миксин. А вот
 содержимого файла `input_large.styl` в собранном `css` нет. Как сделать, чтобы подключался и он?

Создайте `input.deps.yaml` и напишите в нём одну строку:

```
- view: "*"
```

Теперь обновите браузер — стили для селектора `.input_large` придут. И если вы рядом положите файлик
`input_small.styl`, то и его содержимое попадёт в собранный `css`. И для любого другого вью тоже.

Теперь всё почти хорошо. Не нравится только то, что код миксина лежит в файле с именем `input.styl`. Можно так и
оставить, если вас это устраивает. Но если миксинов будет несколько? Доведём объяснение до логического конца.

Создадим для миксинов своё особенное место, чтобы они не мозолили глаза.

1. Создайте папку `blocks/input/_skin` (именно так, с подчеркиванием впереди), перенесите в неё файл `input.styl` и
сразу его переименуйте в `input_skin_base.styl`. Вместо `base` можно написать что вам больше нравится,
а вот конструкция `input_skin` в имени файла - обязательна и неизменна.

2. Откройте его на редактирование и замените для консистентности имя миксина с `input_mixin($height)` на
`input_skin_base($height)`.

3. Сразу же в файле `input_large.styl` замените вызов миксина с `input_mixin($height = 32px);` на
`input_skin_base($height = 32px);`

Слово `skin` - служебное. Папка `_skin` - служебная, специально для хранения миксинов. Мы назвали её так,
потому что такие штуки, как миксины, напоминает нам скины из WinAmp.

Обновите страницу в браузере. Стили исчезли? Исчезли, но не все. Остались только те, чтобы в `input_large.styl`. А код
миксина не попал внутрь селектора `.input_large {...}`. Почему? Потому что вью "Большой Инпут" не знает, где ему брать
код миксина. Он не знает, что нужно открыть папку с названием `_skin`, открыть там файл с именем `input_skin_base.styl`
и в нём искать одноименный миксин.

Обо всём этом сборщику нужно сказать явно. Через зависимость. Нужно сказать, что такие-то вью инпута (а лучше,
что любые вью инпута) зависят от таких-то миксинов. Создадим файл `input_view.deps.yaml`

```
- skin: "*"
  required: true
```

Этот файл надо понимать так: "Любые вью инпутов (это следует из названия `deps`-файла ) зависят от любых миксинов,
находящихся в папке `_skin` (это следует из инструкции `- skin: "*"`). При этом миксины должны попасть в сборку раньше,
чем сами вью (об этом сообщает инструкция `required: true`)"

Обновляем страницу в браузере, стили вернулись.

Теперь в папке `/blocks/input/` мы видим только файлы разных отображений. А миксины спрятаны в папке
`/blocks/input/_skin`. И если нам понадобится, мы можем сделать ещё скинов, сложить в папку `_skin` и звать их из
разных вью. Получается, что папка `_skin` стала своеобразной библиотекой скинов.

Уфф. Про вью и миксины это всё.

Итого, миксин `input_skin_base.styl` хранит всё стили текстового поля, общие для любых отображений блока,
а `input_large.styl` хранит `CSS`-селектор для отображения типа "Большой Инпут".

Нужно другое отображение, например,"Маленький Инпут"? Быстрый путь такой - скопировать и отредатировать:

```
cd blocks/input
cp input_large.styl input_small.styl
```
Отредактировать `input_small.styl`:

Было:
```css
.input_large {
    input_skin_base($height = 32px);

    /* <input> */
    &__control{
        font-size: 16px;

        padding: 0 1em;
    }
}
```

Стало:
```css
.input_small {
    input_skin_base($height = 22px);

    /* <input> */
    &__control{
        font-size: 13px;

        padding: 0 1em;
    }
}
```

Теперь вы понимаете, как мы знаете всё необхожимое, чтобы создавать отображение блока.

Нам осталось исправить одну недоделку в файле `input_skin_base.styl`. Посмотрим, как сейчас выглядит селектор
крестика для очищения поля:

```css
&__clear {
    position: absolute;
    top: 0;
    right: 0;
    z-index: 2;

    width: $height;
    height: $height;
    vertical-align: middle;

    background: url('images/clear-14.png') 50% 50% no-repeat;
    cursor: pointer;
    opacity: (ie != 8) ? 0.3 : 1;

    &:hover {
        background-image: url('images/clear-14-hover.png');
        opacity: 1;
    }

    &._visible {
        display: block;
    }
}
```

Во-первых, у вас нет этих картинок. Возьмите их у нас и скопируйте в проект в папку `/blocks/input/images`.

* [крестик 10 px](http://bevis-ui.github.io/docs/images/input__clear/clear-10.png)
* [крестик 10 px для hover](http://bevis-ui.github.io/docs/images/input__clear/clear-10-hover.png)
* [крестик 14 px](http://bevis-ui.github.io/docs/images/input__clear/clear-14.png)
* [крестик 14 px для hover](http://bevis-ui.github.io/docs/images/input__clear/clear-14-hover.png)

Во-вторых, картинка `clear-14.png` будет применяться для обоих вью - для "Большого" и "Маленького Инпута". Это место
хочется параметризировать: для "Большого Инпута" показывать картинку `clear-14.png`, а для "Маленького Инпута" -
`clear-10.png`. Что делаем? Конечно, новый миксин! :)

Создаём файл `_skin/input_skin_clear.styl` и выносим в него из файла `input_skin_base.styl` только тот код, который
можно параметризировать:

```css
input_skin_clear($size) {
    background: url('images/clear-' + $size + '.png') 50% 50% no-repeat;

    &:hover {
        background-image: url('images/clear-' + $size + '-hover.png');
        opacity: 1;
    }
}
```

Мы понимаем, что параметризировать здесь надо свойство `background-image`, а не составное `background`. Но у нас нет
цели показать вам максимально оптимизированный `CSS`-код. Мы понимаем, что не нам вас учить, как оптимально писать
CSS-код :)

Осталось только позвать этот миксин в каждом вью. Добавим вызовы, каждый со своим размером иконки

В `input_small.styl` передаём параметр `$size = 10`

```css
.input_small {
    input_skin_base($height = 22px);
    input_skin_clear($size = 10);

    /* <input> */
    &__control{
        font-size: 13px;

        padding: 0 1em;
    }
}
```

А в `input_large.styl` передаём `$size = 14`:

```css
.input_large {
    input_skin_base($height = 32px);
    input_skin_clear($size = 14);

    /* <input> */
    &__control{
        font-size: 16px;

        padding: 0 1em;
    }
}
```

Мы столкнулись с неконсистентностью передачи параметров. Почему один параметр передаётся с `px`, а другой без. Но это
не имеет значения для этой страницы документации. `Stylus` позволяет так делать - выбор за вами, как именно писать.

Всё. На этом пора закругляться о стилях.

Получилось как-то многословно. Этот мануал требовал от вас вниматеьности и аккуратности. Но это тема такая, что
отображение блока прочно связано с `html`-разметкой, с файловой системой. Мы старались описывать всё обстоятельно,
по-шагово, без спешки, чтобы вы успевали редактировать с нами синхронно.

Вот, что должно было [у вас получиться](https://github.com/bevis-ui/bevis-stub/tree/input-with-styles/blocks/input).

Сверьте ваш и наш код сейчас, потому что дальше мы будем программировать поведение текстового поля. Если так случилось,
что ваш код отличается от нашего - возможно, вы что-то не сделали во время этого урока. Или, что весьма вероятно, мы
что-то написали не очень понятно или, вообще, с явными ошибками. Если вы нашли ошибку, напишите нам скорее :)

А теперь посмотрим, как мы программируем поведение блока. Нам [сюда](yblock.md).

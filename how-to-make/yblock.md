в разработке

----

Мы с вами проделали уже большой путь — [создали новую страницу](new-page.md),
[создали новый блок и написали простые bt-шаблоны](new-block.md),
[написали к блоку стили и расширили bt-шаблоны](css.md).

У нас есть страница `/pages/test-page/test-page.page.js`, а в ней единственный блок `input`:

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
                {url: options.assetsPath + '.' + params.lang + '.js'}
            ],
            body: [
                {
                    block: 'input',
                    view: 'large',
                    value: 'Привет, Бивис',
                    name: 'loginField',
                    placeholder: 'на сайте'
                }
            ]
        };
    });
}
```

Если проект запустить, то по адресу `http://localhost:8080/test` мы увидим страницу с серым фоном и текстовым полем,
в котором уже введено значение "Привет, Бивис".

Если в браузере заглянуть в `HTML`-код блока, увидим следующую структуру:

```html
<div class="input_large _init" data-block="input">
    <input class="input_large__control" value="Привет, Бивис" name="loginField" placeholder="Инпут на сайте">
    <div class="input_large__clear"></div>
</div>
```

Созданием этого `HTML`-кода занимаются `bt`-шаблоны, которые мы с вами написали в файле `/blocks/input/input.bt.js`.
Стилизацией блока занимается целая система файлов `/blocks/input/input*.styl`.

Самое время запрограммировать поведение блока — cделать блок интерактивным, т.е. отзывчивым на действия пользователя.

Когда мы [создавали новый блок](new-block.md), мы сделали заготовку для клиенсткого программирования. Вот она:

`/blocks/input/input.js`

```javascript
modules.define(
    'input',
    ['inherit', 'block'],
    function (provide, inherit, YBlock) {
        var Input = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);

                console.log(this.getValue());
            },

            getValue: function() {
                return this.getDomNode().val();
            },


            setValue: function(value) {
                this.getDomNode().val(value);
            }
        }, {
            getBlockName: function () {
                return 'input';
            }
        });

        provide(Input);
});
```

Это всего лишь заготовка. Она мало что умеет делать. Да и выглядит странно: какой-то `module.define()`, какой-то
`provide`. Что это?

Мы пишем клиентский `javascript` в виде модулей. Если сказать по-простому, то мы пишем `javascript`-программу и 
помещаем её в специальный контейнер, который называем модулем.

Модульных систем много разных, Бивис использует
[Ymaps Modules](https://github.com/ymaps/modules/blob/master/what-is-this.md), которые придумали и разработали
разработчики из Яндекс.Карт. Пройдите по ссылке, прочитайте.
В основе модулей лежит очень простая идея, и в документации к `Ymaps Modules` простым языком описывается,
зачем модульная система нужна и как её использовать.

Не стали читать? ;) Там чтения на пять минут. Мы всё таки настаиваем, вам понравится.
[Прочитайте](https://github.com/ymaps/modules/blob/master/what-is-this.md).

Прочитали? Тогда вы понимаете, что код внутри анонимной функции - это и есть код,
который программирует поведение вашего `input`. Можно было бы программировать его по старинке,
навешивая события на `DOM`-элементы самим или через jQuery-хелперы. Но мы предпочитаем работать с блоками через
абстракции, чтобы не зависеть от HTML-реализации блока.

Поэтому мы в зависимостях модуля `input` указали два других модуля — `inherit` и `block`.
Первый модуль возвращает функцию `inherit`, которая используется для создания классов и наследования (подробное
описание читайте на странице автора:
[https://github.com/dfilatov/node-inherit](https://github.com/dfilatov/node-inherit).

Второй модуль (`block`) - это `javascript`-класс, который реализует механизмы абстрагирования от `DOM`-узлов. Это 
базовый визуальный блок.

_Все блоки, которые вы создаёте в проекте и которые пользователь может 
увидеть на страницах вашего сайта (мы такие блоки называем визуальными), должны наследоваться от
модуля `block` с помощью модуля `inherit`._

Я употребил слово "наследоваться". Это слово звучит странно применительно к программе на `Javascript`.
Наверное, каждому веб-разработчику известно, что нативного механизма наследования в `Javascript` нет. Но есть
эмуляции, одна из них как раз и есть модуль [inherit](https://github.com/dfilatov/inherit). Документация к этому модулю
написана серьёзно, академически. Здесь же мы попытаемся быстро и по-простому объяснить смысл этого модуля.

Вот вся суть:
```javascript
var B = inherit(A, {});
```

Создаём класс `B`, который наследует методы из класса `A`, и доопределяет своими собственными методами. Эти
дополнительные методы описываются во втором аргументе функции `inherit`. В примере выше никаких методов внутри 
второго параметра не описано, поэтому класс `B` просто отнаследуется от класса `A`.

А уже в этом коде мы с вами создадим класс `Input`, как наследника от класса `Yblock`, и насытим его собственными 
методами `getValue` и `setValue`:

```javascript
var Input = inherit(YBlock, {
    __constructor: function () {
        this.__base.apply(this, arguments);

        console.log(this.getValue());
    },

    getValue: function() {
        return this.getDomNode().val();
    },


    setValue: function(value) {
        this.getDomNode().val(value);
    }
});
```

Метод `__constructor` - особый метод. Если другие методы мы вольны называть как нам нравится, то
именно этот метод должен называться так и никак иначе. Это конструктор класса.

В объектно-ориентированном программировании конструктор класса — это специальная функция, вызываемая при создании
объекта класса. Создадим объект нашего класса `Input`:

```javascript
var myInput = new Input();
```

Когда `javascript`-интерпретатор приступит к выполнению конструкции `new Input()`, он первым делом вызовет функцию
`__constructor`. Можно считать, что это точка входа в вашу программу, описанную в файле `/blocks/input/input.js`

Интерпетатор входит в метод `__constructor` и выполняет всё, что вы там написали.

А написано там пока немного.

Во-первых, инструкция `this.__base.apply(this, arguments);` - её можно не пытаться понять,
просто запомнить и писать в каждом конструкторе каждого вашего блока. 

_Вообще-то это способ из конструктора класса `Input` вызвать конструктор класса `YBlock`._
_То есть, вы описали класс `Input` как наследника от класса `YBlock`. А потом создали экземпляр `myInput`. В тот момент,
когда вы создали объект `myInput`, вызвался его `__constructor`, который вызвал точно такой же `__constructor` 
в `YBlock`._

Если строка `this.__base.apply(this, arguments);` выглядит пугающе, не обращайте внимания, эта строка неинтересная. 
Интересно другое.

С помощью модуля `inherit` мы пишем клиентский `javascript`-код в `ООП`-стиле. Не в виде спагетти-функций, которые друг
друга вызывают в хаотичном порядке, а в виде классов и порожденных ими объектов.

В конструкторе ровно это и происходит - мы вызываем собственный метод `getValue` и результат выводим в консоль браузера:

```javascript
__constructor: function () {
    this.__base.apply(this, arguments);

    console.log(this.getValue());
}
```

Ещё раз о важном. Работа любого модуля начинается с метода `__constructor`. Это точка входа в программу модуля. А уже
 из контруктора мы сами организуем работу модуля как хотим - слушаем события, реагируем на них, может быть, 
 инициализируем какие-то другие модули — что угодно.
   
Например, контрол `input` имеет элемент `clear`, по нажатию на который мы должны очистить поле ввода от введенного 
текста. Алгоритм задачи я бы для себя описал так:

1. Найти элемент, сохранить на него ссылку.
2. Забиндить событие клика на нём и повесить обработчик клика.
3. В обработчике стереть текст из текстового поля.
 
Измените метод `__constructor`, чтобы он выглядел как у меня: 
 
```javascript
__constructor: function () {
    this.__base.apply(this, arguments);

    // 1. Найти элемент, сохранить на него ссылку.
    var clear = this._findElement('clear');
    console.log(clear);
}
```

И тут мы знакомимся с методом `_findElement` — для нас первым методом блока `YBlock` (от которого мы отнаследовали
`Input`). 

Первое, что бросается в глаза — знак нижнего подчеркивания перед именем. Почему `_findElement`, а не `findElement`?
Символ нижнего подчеркивания — это индикатор, что метод приватный... Как бы приватный :) "Как бы", потому что этот  
символ перед именем `javascript`-метода ни на что не влияет. Это в настоящих ООП языках такой метод не может быть  
вызван из другого компонента — сам язык программирования не позволит вам этого сделать технически. Но в `Javascript` 
 такой возможности нет, но очень хочется :)
 
 Хочется отделить публичные методы компонента (которые могут быть вызываны из других компонентов) от приватных 
 методов (которые доступны для вызова только изнутри). А раз технической возможности нет, но сильно хочется, то можно 
 договориться между разработчиками и соблюдать устную договорённость. Мы и договорились — если перед именем метода есть 
 символ нижнего подчеркивания, это метод приватный, относись к нему с почтением :)
   
Так как блок `Input` отнаследован от `YBlock`, то метод `_findElement` у инпута есть. Собственно ради этого метода и 
многих других мы от `YBlock` и хотим наследоваться. Увидеть все методы блока `YBlock` вы 
можете [по этой ссылке.](https://github.com/bevis-ui/bevis-stub/blob/master/core/block/block.js)

А сигнатуру метода `_findElement` мы посмотрим прямо здесь:

```javascript
/**
 * Возвращает первый элемент с указанным именем.
 *
 * @protected
 * @param {String} elementName Имя элемента.
 * @param {HTMLElement|jQuery} [parentElement] Элемент в котором необходимо произвести поиск. Если не указан,
 *                                             то используется результат `this.getDomNode()`.
 * @returns {jQuery|undefined}
 *
 * @example
 * var title = this._findElement('title');
 * title.text('Hello World');
 */
_findElement: function (elementName, parentElement) {
    return this._findAllElements(elementName, parentElement)[0];
}
```
Первый парамет метода - строковое имя элемента, который мы ищем. Обязательный параметр.

Второй параметр — необязательный, о чём нам в `JSDoc`-е говорит имя параметра, заключённое в квадратные скобки 
`[parentElement]` — родительский элемент, в котором необходимо произвести поиск. Обратите внимание, что в этот 
параметр нужно передавать не строковое имя элемента (как было в первом параметре), а именно сам элемент в виде 
`HTMLElement` или в виде `jQuery`, о чём нам сообщает тип параметра
 
```
@param {HTMLElement|jQuery} [parentElement]
```

Метод возвращает `jQuery`, если такой элемент найден, или `undefined`, если такого элемента не существует:
 
```
@returns {jQuery|undefined}
```

Так как этот метод отнаследован нами из `YBlock`, то и вызываем мы его так, словно это метод блока `Input` - через 
ключевое слово `this`. А если отовинуться от монитора и посмотреть на строчку свежим взглядом, то можно прочитать её,
 как предложение на русском языке. 
```javascript
    var clear = this._findElement('clear');
    // У этого блока найди элемент 'clear' и сохрани его в локальной переменной. 
}
```
Понятно ведь, правда?


Ниже строки с вызовом `_findElement` мы добавили вывод в консоль и теперь посмотрим, что воернёт метод в нашей 
программе. Обновляем `http://localhost:8080/test` (или открываем, если закрыли) и видим в консоли браузера что-то  
подобное:

```javascript
{
    0: 'div.input_large__clear',
    context: 'div.input_large__clear',
    length: 1,
    __proto__: x[0]
}
``` 
Это `jQuery`-представление нашего элемента `clear`. Отлично, значит элемент нашли и сохранили в локальную переменную 
внутри метода `__constructor`.

Двигаемся дальше, и будем слушать событие `click` на крестике:
 
```javascript
__constructor: function () {
    this.__base.apply(this, arguments);

    // 1. Найти элемент, сохранить на него ссылку.
    var clear = this._findElement('clear');
    
    // 2. Забиндить событие клика на нём и повесить обработчик клика.
    this._bindTo(clear, 'click', function () {
        console.log('Крестик нажат');
    });
}
```

Знакомимся со следующим методом — `_bindTo`. Сигнатуру его тоже посмотрим здесь:
```javascript
/**
 * Добавляет обработчик события `event` объекта `emitter`. Контекстом обработчика
 * является экземпляр данного блока. Обработчик события автоматически удалится при вызове
 * `Block.prototype.destruct()`.
 *
 * @protected
 * @param {jQuery|Block} emitter
 * @param {String} event
 * @param {Function} callback
 * @returns {Block}
 *
 * @example
 * var View = inherit(Block, {
 *     __constructor: function (model) {
 *         this.__base();
 *
 *         var hide = this._findElement('hide');
 *         this._bindTo(hide, 'click', this._onHideClick);
 *
 *         this._bindTo(model, 'change-attr', this._onAttrChange);
 *     }
 * });
 */
_bindTo: function (emitter, event, callback) {
    this._eventManager.bindTo(emitter, event, callback);
    return this;
}
```
Первый параметр — элемент или блок, на котором слушаем событие.

Второй параметр — строковое имя события.

Третий — функция-обработчик события. Вызовется, когда событие случится.

Тоже приватный метод. И тоже читается, как предложение на русском языке.

```javascript
    this._bindTo(clear, 'click', function () {
        console.log('Крестик нажат');
    });
    // У этого блока привяжи элемент clear к событию 'click', и когда оно случится, вызови функцию-обработчик.
}
```

Обновите страницу в браузере и нажмите мышкой на крестик. В консоли видим сообщение: "Крестик нажат"? Прекрасно.

В `JSDoc`-е к методу `_bindTo` есть примеры его вызова. Видно, что в примерах третий параметр — это не сама 
функция-обработчик, а только ссылка на неё. Сделаем у себя так же:

1. Отредактируйте третий параметр в методе `_bindTo`

2. Ниже конструктора добавьте метод `_onClearClicked` и напиши в нём какой-то новый `console.log`, чтобы убедиться, 
что изменения вступили в силу: 

Было:
```javascript
__constructor: function () {
    this.__base.apply(this, arguments);

    // 1. Найти элемент, сохранить на него ссылку.
    var clear = this._findElement('clear');
    
    // 2. Забиндить событие клика на нём и повесить обработчик клика.
    this._bindTo(clear, 'click', function () {
        console.log('Крестик нажат');
    });
}
```

Стало:

```javascript
__constructor: function () {
    this.__base.apply(this, arguments);

    // 1. Найти элемент, сохранить на него ссылку.
    var clear = this._findElement('clear');

    // 2. Забиндить событие клика на нём и повесить обработчик клика.
    this._bindTo(clear, 'click', this._onClearClicked);
},

_onClearClicked: function () {
    console.log('_onClearClicked: Крестик нажат');
},
```

Проверям. Если в консоли появилось сообщение: "_onClearClicked: Крестик нажат", значит успех.

Мы только что создали собственный приватный метод блока `Input`. Именно в нём мы будем очищать текстовое поле ввода.

Кстати, у нас же есть метод `setValue`, который предназначен для установки значения в текстовое поле. Воспользуемся 
им, только будем устанавливать пустую строку.

```javascript
_onClearClicked: function () {
    this.setValue('');
},
```

Проверяем, кликаем на крестик... и ничего не происходит. Это почему вдруг?

Ну, конечно, вот же ошибка. В методе `setValue` написано:
```javascript
setValue: function(value) {
    this.getDomNode().val(value);
}
```

Если я правильно помню, функция `val` — это `jQuery` функция, которая отлично устанавливает значение в текстовые поля 
формы. Проблема в том, что тот объект, который возвращает некий метод `this.getDomNode()`, не является `<input>`.  А 
что это такое вообще — `this.getDomNode()`?

Знакомьтесь, новый метод `YBlock` — метод `getDomNode`:
```javascript
/**
 * Возвращает DOM-элемент данного блока.
 *
 * @returns {jQuery}
 */
getDomNode: function () {
    return this._node;
}
```

Давайте проверим. Добавим `console.log` в метод `setValue`:

```javascript
setValue: function(value) {
    console.log(this.getDomNode());

    this.getDomNode().val(value);
}
```

В консоли вдруг видим `jQuery`-представление блока целиком, но никак не представление элемента `<input>`:

```javascript
{
    0: 'div.input_large._init',
    context: 'div.input_large._init',
    length: 1,
    selector: '',
    __proto__: x[0]
}
```
Получается, что мы пытаемся вызвать `jQuery`-функцию `val()` от DOM-элемента `<div class="input_large">...</div>`?


Ну да, мы же в файле [css.md](css.md) начинали с того, что блок 'input' был представлен только `DOM`-элементом `<input 
class="input_large">` и тогда же мы создавали `js`-заготовку, в которой описали метод `setValue()`. В тот момент этот 
метод работал правильно, потому что `this.getDomNode()` возвращал `jQuery`-представление текстового поля формы, и  
функция `val` работала корректно.
 
А чуть позже мы усложнили `HTML`-структуру блока, но `js`-код поправить забыли.

Вот как выглядит `HTML` блока сейчас:
```html
<div class="input_large _init" data-block="input">
    <input class="input_large__control" value="Привет, Бивис" name="loginField" placeholder="Инпут на сайте">
    <div class="input_large__clear"></div>
</div>
```

Следовательно, метод `setValue` (а так же `getValue`) должны работать не с `this.getDomNode()`, а с `DOM`-элементом 
поля формы. Текстовое поле формы у нас представлено тегом `<input class="input_large__control">` (см. `HTML` выше), то 
есть в терминах `BEViS` текстовое поле формы — это элемент `control` у блока `input-large` (см. атрибут 
`class="input_large__control"`).
 
Перепишем методы, чтобы значение извлекалось/устанавливалось в/из текстового поля формы:
```javascript
getValue: function() {
    var control = this._findElement('control');
    return control.val();
},


setValue: function(value) {
    var control = this._findElement('control');
    control.val(value);
}
```

Всё, теперь работает, текст удаляется, вместо него возникает плейсхолдер.

Сделаем код опрятнее, удалим диблирование, допишем комментарии. У меня это получилось так:
```javascript
modules.define(
    'input',
    ['inherit', 'block'],
    function (provide, inherit, YBlock) {
        var Input = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);

                // Находим все элементы блока
                this._clear = this._findElement('clear');
                this._control = this._findElement('control');

                // Обрабатываем клик по крестику
                this._bindTo(this._clear, 'click', this._onClearClicked);
            },

            /**
             * Очищает поле ввода
             */
            _onClearClicked: function () {
                this.setValue('');
            },

            /**
             * Получает значение из текстового поля
             *
             * @returns {String | undefined}
             */
            getValue: function() {
                return this._control.val();
            },

            /**
             * Устанавливает значение к текстовое поле
             * 
             * @param {String} value Переданное значение
             */
            setValue: function(value) {
                this._control.val(value);
            }
        }, {
            getBlockName: function () {
                return 'input';
            }
        });

        provide(Input);
});
```

Всё работает. Осталось только по-тихоньку дописывать нужную функциональность.

Посмотрим на эскиз дизайнера ещё раз.

<img src="http://bevis-ui.github.io/docs/images/input-design.png" width="557" height="539" />

Что нужно реализовать ещё:

* деактивацию контрола, чтобы не кликался; и обратную ей активацию, 
* установку фокуса, чтобы вокруг появлялось жёлтое свечение,
* наверное, стоит сделать обработку нажатия клавиши `Enter`, чтобы пользователь мог отправлять данные, не уходя из поля 

Для деактивации и для фокуса мы в стилях создавали специальные состояния `disabled` и `focused`, которые визаульно 
делают блок соответственно неактивным или в фокусе.

Состояния в `BEViS` ставятся с помощью приватного метода `_setState`. Вот его сигнатура:

```javascript
/**
 * Устанавливает CSS-класс по имени и значению состояния.
 * Например, для блока `button` вызов `this._setState('pressed', 'yes')`
 * добавляет CSS-класс с именем `pressed_yes`.
 *
 * С точки зрения `BEM` похож на метод `setMod`, но не вызывает каких-либо событий.
 *
 * @protected
 * @param {String} stateName Имя состояния.
 * @param {String|Boolean} [stateVal=true] Значение.
 *                                         Если указан `false` или пустая строка, то CSS-класс удаляется.
 * @returns {Block}
 */
_setState: function (stateName, stateVal) {
    // ...
}
```

Состояния могут быть однословные, как здесь нам нужно — просто `_disabled` или просто `_focused`. А могут быть 
двусловные, например `_pressed_yes`. Этом метод позволяет создавать и те и другие, и даже удалять состояние. Но для 
удаления есть симметричный метод `_removeState` (приводить его сигнатуру здесь не буду, 
[можно посмотреть](https://github.com/bevis-ui/bevis-stub/blob/master/core/block/block.js) самому, когда возникнет 
необходимость).

Я написал эту функциональность, гляньте. Мне кажется, объяснять там уже особо нечего, вы всё поймёте и без подсказок 

```javascript
modules.define(
    'input',
    ['inherit', 'block'],
    function (provide, inherit, YBlock) {
        var Input = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);

                // Находим все элементы блока
                this._clear = this._findElement('clear');
                this._control = this._findElement('control');

                // Обрабатываем клик по крестику
                this._bindTo(this._clear, 'click', this._onClearClicked);

                // Слушаем события, которые выбросил <input>
                this._bindTo(this._control, 'keypress', this._onKeyPressed);
                this._bindTo(this._control, 'focus', this.focus);
                this._bindTo(this._control, 'blur', this.blur);
            },

            /**
             * Очищает поле ввода
             */
            _onClearClicked: function () {
                if (this.isEnabled()) {
                    this.setValue('');
                    this.focus();
                }
            },

            /**
             * Устанавливает значение к текстовое поле
             *
             * @param {String} value Переданное значение
             * @returns {Input}
             */
            setValue: function(value) {
                this._control.val(value);
                return this;
            },

            /**
             * Получает значение из текстового поля
             *
             * @returns {String | undefined}
             */
            getValue: function() {
                return this._control.val();
            },

            /**
             * Устанавливает фокус на поле ввода.
             *
             * @returns {Input}
             */
            focus: function () {
                if (this.isEnabled()) {
                    this._setState('focused');
                    this._control.focus();
                }
                return this;
            },

            /**
             * Удаляет фокус с поля ввода.
             *
             * @returns {Input}
             */
            blur: function () {
                this._removeState('focused');
                this._control.blur();
                return this;
            },

            /**
             * Возвращает `true`, если поле ввода активно.
             *
             * @returns {Boolean}
             */
            isEnabled: function () {
                return !this._getState('disabled');
            },

            /**
             * Деактивирует поле ввода.
             *
             * @returns {Input}
             */
            disable: function () {
                if (this.isEnabled()) {
                    this.blur();
                    this._control.attr('disabled', 'disabled');
                    this._setState('disabled');
                }
                return this;
            },

            /**
             * Активирует поле ввода.
             *
             * @returns {Input}
             */
            enable: function () {
                if (!this.isEnabled()) {
                    this._control.removeAttr('disabled');
                    this._removeState('disabled');
                }
                return this;
            },

            /**
             * В инпуте нажали кнопку "Enter"
             *
             * @param {Event} e
             */
            _onKeyPressed: function (e) {
                if (e.keyCode === 13) {
                    this.emit('input-submitted', {
                        value: this.getValue()
                    });
                }
            }

        }, {
            getBlockName: function () {
                return 'input';
            }
        });

        provide(Input);
});
```

Я уверен, вы всё поняли, даже это:
```javascript
_onKeyPressed: function (e) {
    if (e.keyCode === 13) {
        this.emit('input-submitted', {
            value: this.getValue()
        });
    }
}
```

Мы на инпуте слушаем событие `keypressed` и ловим момент, когда нажали клавишу `Enter`. После этого генерим кастомное 
событие `input-submitted` (я его сам придумал) с помощью ещё одного метода, который унаследован от `YBlock`, а тот в 
свою очередь унаследовал его от класса `EventEmitter`. Документацию к этому классу и к его методам для работы с  
событиями [можно прочитать здесь](https://github.com/bevis-ui/bevis-stub/blob/master/core/event-emitter/event-emitter
.js). 

Метод `emit` позволяет "выкинуть" любое произвольное событие. А тот блок, кому важно узнать об этом событие, на это 
событие подписывается, ждёт, а потом слышит его и как-то реагирует. Я не сразу понял, как кастомные события работают. 

Кассир в Макдональдсе выбрасывает вверх руку и кричит в воздух "Свободная касса!". И кто проворнее, кому больше надо,
 тот спешит к кассе и получает от кассира бургер.
 
 Здесь так же. Метод `emit` - тот же кассир. Он выбрасывает в воздух клич, в нашем случае он кричит, что, мол, на 
 инпуте произошёл так называемый `submit`. И тот, кто ждёт такой клич (кто подписался на событие `input-submitted`), 
 получает от метода какие-то данные из второго параметра. В нашем случае получает объект `{value: this.getValue()}`. 
  Ясное дело, в объекте будет не функция `this.getValue()`, а результат её работы, то есть текст, введённый в поле.
 
 Ну, в самом деле, мы же не хотим (не должны хотеть) по нажатию на `Enter` отправлять данные прямо из кода инпута?  
 Инпуту в идеале не нужно знать ни куда отправлять, ни в каком виде эти данные должны идти. Мы вообще-то с вами даже не 
 какой-то конкретный инпут только что запрограммировали — мы описали Класс `Input`, на основе которого можно сделать 
  множество экземпляров этого класса, и каждый из них будет иметь свои поля `this._clear`, `this._control` и все 
  описанные выше методы.
  
  Поэтому по нажатию на `Enter` самое лучше, что инпут может сделать - это крикнуть в воздух - я сделал то-то, 
  заберите у меня данные. И пусть, кому надо, тот и забирает.
  
  А кому обычно нужны данные? В старом вебе, когда мы не умели использовать XMLHTTPRequest, мы писали в `HTML` формы,
   которые были представлены тегом `<form action="path-to-server-script">`. И по клику на кнопку сабмита, браузер 
   собирал данные из полей формы и отправлял по адресу, указанному в атрибуте `action`. 
   
Сейчас мы всё больше пишем `javascript`-приложения, данные с полей форм собираем с помощью `js` и отправляем их на 
сервер с помощью XMLHTTPRequest. Это я к тому, что тег `<form>` нам особо и не нужен теперь. Разве что в качестве  
фоллбека (а вдруг ещё есть пользователи, которые отключают `Javascript` в браузере?). А что нам точно нужно - так это 
js-компонент, который будет исполнять роль формы. Именно он будет подписываться на кастомное событие 
`input-submitted`, получать данные из него и дальше что-то делать (сейчас не суть важно что).
 
Создадим такой?

```
make block
# Введите имя блока: form
```

Посмотрите, по адресу `blocks/form` появился новый блок. Его bt-шаблон уныл и в стилях ни строчки осмысленного 
кода. Пусть так и будет. Нам не нужны ни какие-то особенные шаблоны, ни стили. Нам интересно другое. И это так
 прекрасно, что я тороплюсь вам об этом рассказать.
 
Давайте отредактируем `pages/test-page/test-page.page.js`, чтобы у вас получилось как у меня:

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
                {url: options.assetsPath + '.' + params.lang + '.js'}
            ],
            body: [
                {
                    block: 'form'
                }
            ]
        };
    });
};
```

Мы выкинули инпут и задекларировали форму. Можете быстро глянуть в браузере - там ничего нет. Зависимость не добавили!

Добавим зависимость к `form` в `pages/test-page/test-page.deps.yaml`, чтобы у вас получилось как у меня:

```
- page
- block: block
  elem: auto-init
- input
- form
```

Проверяем в браузере - да, теперь "форма" отредерилась.

Чтобы клиентский `js`-код работал, в `bt`-шаблон добавим строку `ctx.enableAutoInit();`

```javascript
module.exports = function (bt) {
    bt.match('form', function (ctx) {
        ctx.enableAutoInit();
        ctx.setTag('span');
        ctx.setContent('Содержимое блока');
    });
};
```
 
А теперь мы налету будем генерить `input`!
 
 Открываем `blocks/form/form.js` и пишем:

```javascript
modules.define(
    'form',
    ['inherit', 'block', 'input'],
    function (provide, inherit, YBlock, Input) {
        var form = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);

                // Создаём инпут
                this._greetingInput = new Input();

                // Рендерим форму
                this._render();
            },

            /**
             * Рендерит все контролы в форме
             */
            _render: function () {
                var form = this.getDomNode();
                var greeting = this._greetingInput.getDomNode();

                greeting.appendTo(form);
            }      
        }, {
            getBlockName: function () {
                return 'form';
            }
        });

        provide(form);
});
```
Смотрите внимательно, я в зависимостях указал модуль `input` (3-я и 4-я строки)

А потом в конструкторе формы очень красиво, коротко, понятно сказал:
```javascript
// Создаём инпут
this._greetingInput = new Input();
```

Как только я это сказал, тут же (в памяти браузера) возник новый экземпляр класса `Input` и сохранился в 
поле `_greetingInput`. 

А как только я "заапендил" этот экземпляр в `DOM` дерево, он тут же инициализовался и готов к работе.

И в браузере отобразился инпут, который умеет слушать клики на крестике, нажатие клавиши `Enter` и всё другое.

Красиво, коротко, понятно: `new Input()` и готово.

Если нужно наполнить поле значением, делаем это через публичные методы класса `Input`:

```javascript
// Создаём инпут
this._greetingInput = new Input();
this._greetingInput.setValue('Привет, Бивис!');
```

Можно тут же установить фокус или деактивировать поле, в-общем, делать всё то, для чего
 написаны публичные методы класса `Input`:
```javascript
this._greetingInput.focus();
this._greetingInput.disable();
```

И наконец, мы можем подписаться на произвольные события, генерируемые классом `Input`.

```javascript
// Создаём инпут
this._greetingInput = new Input();
this._greetingInput.on('input-submitted', this._onInputSubmitted, this);
```

Подписка на кастомные события происходит с помощью метода `on()`, который вызывается от объекта, на котром мы хотим 
услышать событие. Этот метод есть у каждого визуального блока, потому что он наследован от класса `EventEmitter`.
 
Мметод `on` тоже читается, словно на человеческом языке: 
```javascript
this._greetingInput.on('input-submitted', this._onInputSubmitted, this);
// Этот инпут-контрол слушает событие `input-submitted`, и когда оно случится, вызовется метод `_onInputSubmitted`
```

Что будет происходить в обработчике - не знаю. У меня обработчик пока только вываливает в консоль сообщение о том, что 
обработчик вызыван. У вас там наверняка будет валидация данных во всех полях формы и отправка данных куда-то в бекенд.

```javascript
modules.define(
    'form',
    ['inherit', 'block', 'input'],
    function (provide, inherit, YBlock, Input) {
        var form = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);

                // Создаём инпут
                this._greetingInput = new Input();
                this._greetingInput.on('input-submitted', this._onInputSubmitted, this);

                // Рендерим форму
                this._render();
            },

            _onInputSubmitted: function () {
                console.log('Форма поймала событие от Input');
            },

            /**
             * Рендерит все контролы в форме
             */
            _render: function () {
                var form = this.getDomNode();
                var greeting = this._greetingInput.getDomNode();

                greeting.appendTo(form);
            }
        }, {
            getBlockName: function () {
                return 'form';
            }
        });

        provide(form);
});
```

Мне не нравится, что модуль с формой занимается `append`-ом инпута внутрь себя. Мне кажется, форма этим заниматься не
 должна. Инпут сам должен рендерить себя — пусть рендеринг инпута происходит в модуле `Input`, а не в модуле `Form`. 
  Не могу аргументированно доказать, почему именно такой подход я считаю правильным, но я ему следую. "Я художник — 
  я так вижу" (с) 
  
Есть по-сильнее аргумент. Если инпут будет уметь сам себя рендерить, тогда он может оказаться в любом другом блоке, а
  не только в _этой_ форме — любой блок сможет создать инстанс Инпута и не заботиться о том, что нужно ещё 
написать некий метод, чтобы встроить этот инпут в себя. Хочется, чтобы инпут умел принимать параметр со ссылкой на 
"родительскую" `DOM`-ноду, куда инпуту нужно встроиться.  

Для этого нам пригодится возможность в конструктор класса передавать параметры.

```javascript
modules.define(
    'form',
    ['inherit', 'block', 'input'],
    function (provide, inherit, YBlock, Input) {
        var form = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);

                var formDomNode = this.getDomNode();

                // Создаём инпут
                this._greetingInput = new Input({
                    parentNode: formDomNode
                });

                this._greetingInput.on('input-submitted', this._onInputSubmitted, this);
            },

            _onInputSubmitted: function () {
                console.log('Форма поймала событие от Input');
            }
        }, {
            getBlockName: function () {
                return 'form';
            }
        });

        provide(form);
});
```

1. Мы удалили метод `render()`. Он не нужен.

2. В конструкторе формы мы создали локальную переменную `formDomNode`, в которую получили `DOM`-представление формы.

3. В создаваемый экземпляр класса `Input` мы передали произвольный параметр `parentNode`, в значении которого 
оказалось `DOM`-представление формы. Имя `parentNode`, как и `formDomNode` - абсолютно произвольные. 

Было:
```javascript
// Создаём инпут
this._greetingInput = new Input();
```

Стало:
```javascript
// Создаём инпут
this._greetingInput = new Input({
    parentNode: formDomNode
});
```

Код модуля `Form` стал меньше и понятнее. В нём не осталось никакой лишней логики, есть только нужное для формы: 
* создаёт экземпляр инпута
* если инпут выкинет событие `input-submitted` — форма отреагирует каким-то образом. 

А код модуля `Input` нужно дописать:
* принимать в конструкторе параметр `parentNode`, в котором хранится `DOM`-представление формы
* "заппендить" инпут внутрь формы 

Смотрите, что я изменил в модуле `input.js`:

Было:
```javascript
__constructor: function (params) {
    this.__base.apply(this, arguments);

    // Находим все элементы блока
    this._clear = this._findElement('clear');
    this._control = this._findElement('control');

    // Обрабатываем клик по крестику
    this._bindTo(this._clear, 'click', this._onClearClicked);

    // Слушаем события, которые выбросил <input>
    this._bindTo(this._control, 'keypress', this._onKeyPressed);
    this._bindTo(this._control, 'focus', this.focus);
    this._bindTo(this._control, 'blur', this.blur);
},
```

Стало:
```javascript
__constructor: function (params) {
    this.__base.apply(this, arguments);

    // Получаем параметр с DOM-нодой родителя.
    this._parentNode = params.parentNode;

    // Отредери себя!
    this._render();
    
    // Находим все элементы блока
    this._clear = this._findElement('clear');
    this._control = this._findElement('control');

    // Обрабатываем клик по крестику
    this._bindTo(this._clear, 'click', this._onClearClicked);

    // Слушаем события, которые выбросил <input>
    this._bindTo(this._control, 'keypress', this._onKeyPressed);
    this._bindTo(this._control, 'focus', this.focus);
    this._bindTo(this._control, 'blur', this.blur);
},

/**
 * Рендерит инпут внутри родительской DOM-ноды this._parentNode
 */
_render: function () {
    this.getDomNode().appendTo(this._parentNode);
},
```

Так можно передавать любые параметры из одного блока в другой. Так блоки могут обмениваться данными. 

Но это ещё не всё. Ведь можно в конструктор инпута передавать параметры, как мы это делали в bt-шаблоне.

Помните, мы в `test-page.page.js` когда-то декларировали статический `btjson`, а позже от него отказались?
```javascript
    body: [
        {
            block: 'input',
            view: 'large',
            value: 'Привет, Бивис',
            name: 'loginField',
            placeholder: 'Инпут на сайте'
        }
    ]
```

Точно такой же статический `btjson` можно передавать в качестве параметров при создании экземпляра нашего Инпута.

Возвращаемся в модуль `form.js`

Было:
```javascript
// Создаём инпут
this._greetingInput = new Input({
    parentNode: formDomNode
});
```

Стало:
```javascript
// Создаём инпут
this._greetingInput = new Input({
    value: 'Привет, Бивис',
    name: 'loginField',
    placeholder: 'Инпут на сайте',

    parentNode: formDomNode
});
```

И обновите страницу в браузере.
 
Внезапно, в текстовом поле оказалось значение "Привет, Бивис". А если фаербагом посмотреть в `HTML`, увидим в атрибуте 
`name` значение "loginField", а в `placeholder` значение "Инпут на сайте". 

И при этом инпут отрендерился внутри формы (сам!), на нёго повесились все обработчики событий.
И при этом форма слушает его кастомные события.

Я очень люблю `BEViS` за это. Оказывается, `bt`-шаблоны работают и на серверной стороне и в браузере.

То есть, если надо создать блок на серверной стороне, мы в теле страницы `test-page.page.js` декларируем блок и 
описываем нужные опции. Вот фрагмент страницы `test-page.page.js` со статической декларацией блока `input`.
```javascript
body: [
    {
        block: 'input',
        view: 'large',
        value: 'Привет, Бивис',
        name: 'loginField',
        placeholder: 'Инпут на сайте'
    }
]
```
И знаем, что сервер отправит браузеру полностью готовый код блока в виде набора `HTML`-тегов со 
всеми нужными данными для отображения.

Если надо создать не статический блок (когда все данные для блока известны), а динамический (когда часть данных может
  быть известна только во время работы приложения в браузере) из клиентского `javascript`, то нам нужно выполнить два
    шага:

1. Создать файл-болванку для клиентского `javascript`-модуля (можно сделать с помощью команды `make block`)
```javascript
modules.define(
    'input',
    [
        'inherit',
        'block'
    ],
    function (
        provide,
        inherit,
        YBlock
    ) {
        var Input = inherit(YBlock, {
            __constructor: function () {
                this.__base.apply(this, arguments);
            }
        }, {
            getBlockName: function () {
                return 'input';
            }
        });

        provide(Input);
    }
);
```
Этой болванки абсолютно достаточно, чтобы выполнить второй шаг.

2. В каком-то другом модуле (например `form.js`) вызвать конструктор класса `Input` с `bt`-опциями, которые можно 
заполнить динамическими данными:

```javascript
// Создаём инпут
var currentValue = 'Привет, Бивис'; // Здесь, например, вызов некоей функции, которая вернёт некое значение для поля.
var currentName = 'loginField'; 

this._greetingInput = new Input({
    value: currentValue,
    name: currentName,
    placeholder: 'Инпут на сайте',
});
```

В результате работы конструктора `new Input` в браузере будет создан `HTML` инпута прямо налету. И получить его можно
 с помощью метода `getDomNode()`: `this._greetingInput.getDomNode()`.
  
А дальше этот `HTML` нужно "заппендить" в `DOM`-дерево страницы каким-то образом - на ваше усмотрение. Наши  
предпочтения на этот счёт я описал выше.

Это мы говорили про первый способ генерации блока на стороне браузера.

А второй способ — не полагаться на `bt`-шаблоны, а написать в `input.js` специальные методы-сеттеры (от слова 'to 
set' - устанавливать), с помощью которых можно установать необходимые значения. Мы с такого метода начинали с вами: 
это метод `setValue()`. И в каком-то другом модуле (например `form.js`) после вызова конструктора класса `Input` 
вызывать методы-сеттеры, чтобы заполнить инпут данными:
 
```javascript
this._greetingInput = new Input();
this._greetingInput.setValue('Привет, Бивис!');
```

Мы в работе пользуемся всеми тремя способами одновременно. То, что можно сгенерить статически - генерим на сервере. 
Если какие-то данные для генерации блока мы можем получить только в браузере, мы генерим блок динамически.

----



# YBlock

Базовый класс для написания блоков, содержит общее для всех блоков поведение:

* Методы для размещения блока в нужном фрагменте `DOM`-дерева.
* Базовый конструктор и деструктор.
* Инициализация/деинициализация блоков на переданном фрагменте `DOM`-дерева.
* Интерфейс для отложенной (`live`) инициализации.
* Статические метод класса

----



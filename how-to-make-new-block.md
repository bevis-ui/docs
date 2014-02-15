# Разработка BEViS-блоков

Блок — это самостоятельная единица интерфейса сайта. Например, кнопка или поле ввода.
Никому не нужно знать, как устроен блок внутри, но те, кто будет использовать блок снаружи,
им хочется, чтобы внешне блок был красивый, простой и логичный.

## Требования

Мы начинаем разрабатывать новые блоки тогда, когда они нужны для выполнения каких-либо требований.
Например, мне нужно поле для ввода email'а. Когда пользователь вводит данные и нажимает кнопку "Отправить",
то нужно данные из поля ввода взять и отправить на сервер.

* Поле ввода должно возвращать введенные данные.

В Яндексе принят определенный фирменный стиль, и поле ввода должно соответствовать этому стилю.

* Поле ввода должно быть выполнено в фирменном стиле.

## Проектирование

Полезно спроектировать блок. Искусство проектирования может показаться нетривиальным. 
Проектирование — это то, чем мы занимаемся в процессе разработки. Явно или не явно.
На бумаге, в виде диаграмм или просто в виде набора мыслей мы всегда изображаем план своей работы.

Например, можно поиграть с UML-диаграммой. 

Синаксис PlantUML:

```puml
class Input {
    +getValue(): String
}
```

Отправляем в http://www.plantuml.com/plantuml/form :

<img src="http://www.plantuml.com/plantuml/img/Iyv9B2vMyCmhA2rHgEPI00BjzDIIiCISqbGDJIk5u9AYpBnqhbe0" />

Дополним интерфейс инпута до консистентного состояния, добавив сеттер:

<img src="http://www.plantuml.com/plantuml/img/Iyv9B2vMyCmhA2rHgEPI00BjzDIIiCISqbGDJIk5u9AYpBnqY7WnJBmCHCBaDBbg0G00" />

Не будем раньше времени усложнять блок. Он получился простым и расширяемым в будущем.

* Подробнее о PlantUML можно почитать здесь: http://plantuml.sourceforge.net/
* Здесь можно быстро сгенерировать картинки на основе PlantUML-кода: http://www.plantuml.com/plantuml/form

## Директория с блоком

Теперь следует выбрать директорию для того, чтобы разместить блок в дереве проекта.
Если речь о библиотеке `islets`, то визуальные блоки мы размещаем в директории `blocks/common`.
В вашем случае структура директорий может быть отличающейся, более подходящей требованиям самого проекта.
Создаем директорию. Например, `blocks/common/y-input`.
Префикс `y-` мы используем только для блоков внутри библиотеки `islets` для того,
чтобы избежать возможные конфликты в будущем. В проектах не следует использовать префикс `y-`.

## Шаблон

Реализацию блока я обычно начинаю с шаблона.
Таким образом, я могу вставить блок на страницу и разрабатывать, визуально наблюдая результат.

Что нам нужно от шаблона?

* Отображение в браузере в виде тега `input`.
* Возможность задать значение с помощью параметров.

Попробуем набросать первый вариант.
Создаем файл `input.bt.js` в директории `blocks/common/input` со следующим содержимым:

```javascript
module.exports = function (bt) {
    bt.match('input', function (ctx) {
        ctx.setTag('input');
        ctx.setAttr('value', ctx.getParam('value'));
    });
};
```

Да, шаблоны описываются в виде CommonJS (nodejs) модулей.
Шаблоны обрабатываются с помощью BT. Подробнее о BT можно почитать здесь: https://github.com/enb-make/bt

Используем блок на странице:

```javascript
{
    block: 'input',
    value: 'Hello'
}
```

Отлично, все работает.

Теперь, если мы пишем блок для `islets`, то обязательно надо спрятать шаблон в какой-нибудь `view`.
По умолчанию, в `islets`-блоках используется `view` со значением `islet`:

```javascript
module.exports = function (bt) {
    bt.setDefaultView('y-input', 'islet');
    bt.match('y-input_islet', function (ctx) {
        ctx.setTag('input');
        ctx.setAttr('value', ctx.getParam('value'));
    });
};
```

В таком виде блок подойдет и для библиотеки.

## Стилизация

Для того, чтобы оформить блок, создадим файл `input.styl` в директории `blocks/common/input` со следующим содержимым:

```css
.input {
    border: #eee 1px solid;
}
```

Теперь наш блок обзавелся и оформлением.

Следует отметить, что мы используем CSS-препроцессор `stylus`. Подробнее о нем: http://learnboost.github.io/stylus/

## Поведение

Осталось наградить блок поведением.
Если поведение требуется для блока, необходимо установить флаг инициализации в шаблоне:

```javascript
module.exports = function (bt) {
    bt.match('input', function (ctx) {
        ctx.enableAutoInit();
        ctx.setTag('input');
        ctx.setAttr('value', ctx.getParam('value'));
    });
};
```

Теперь можно перейти и к JS-реализации блока. Создадим файл `input.js` в директории `blocks/common/input`.
Для начала следует декларировать модуль `input` в модульной системе:

```javascript
modules.define(
    'input', // имя модуля
    [], // зависимости
    function( // коллбек, который будет выполнен при удовлетворении зависимостей и загрузке модуля
        provide // функция, в которую необходимо передать содержимое модуля
    ) {
        provide(null); // передадим хоть что-нибудь
    }
);
```

Нам надо отнаследоваться от `YBlock` для того, чтобы наш класс стал визуальным блоком.
Наследоваться будем с помощью механизма `inherit`:

```javascript
modules.define(
    'input',
    [
        'inherit', // зависимость от inherit
        'y-block' // зависимость от YBlock
    ],
    function (
        provide,
        inherit, // получаем функцию inherit
        YBlock // получаем блок YBlock
    ) {
        var Input = inherit(YBlock, {
            // инстанс-методы
        }, {
            // статические методы
        
            // при наследовании от YBlock, необходимо переопределить статический метод getBlockName
            getBlockName: function () {
                return 'input'; // вернуть необходимо имя блока
            }
        });
        provide(Input);
    }
);
```

Осталось добавить необходимое поведение:

```javascript
modules.define('input', ['inherit', 'y-block'], function (provide, inherit, YBlock) {
    var Input = inherit(YBlock, {
        /**
         * @returns {String}
         */
        getValue: function() {
            return this.getDomNode().val();
        },
        /**
         * @param {String} value
         */
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

Осталось убедиться, что все работает.

* Подробнее о `YBlock`: https://github.yandex-team.ru/pages/islets/islets/blocks/blocks.ru.html#y-block
* Подробнее об `inherit`: https://github.com/dfilatov/inherit

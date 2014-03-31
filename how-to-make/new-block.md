в разработке

* Упростить текст, сейчас сложно воспринимать
* Ответить, как обращаться за данными и передавать в блок?

----

На примере блока `input`. Создадим такой блок.

# Проектирование

Полезно спроектировать блок. Искусство проектирования может показаться нетривиальным и избыточным.
Но проектирование — это то, чем мы занимаемся в процессе разработки осознанно или не осознанно,
хотим мы того или не хотим. Делаем мы это в голове в виде мыслей или на бумаге в виде схем и диаграмм,
но каждый из нас этим занимается. Займёмся и сейчас.

Изобразим UML-диаграмму будущего блока. Это необязательно для вас, но мы сделаем,
чтобы явно показать ход наших мыслей.

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

На этом остановимся, не будем раньше времени усложнять блок. Блок мы задумали простым: значение из блока можно
получить, и можно передать значение в блок, на этом этапе нам от блока больше ничего не надо.

----

* Подробнее о PlantUML можно почитать здесь: http://plantuml.sourceforge.net/
* Здесь можно быстро сгенерировать картинки на основе PlantUML-кода: http://www.plantuml.com/plantuml/form

----

# Создадим файлы для блока

Запустить команду и ответить на вопрос:
```shell
make block
# Введите имя блока: <ИМЯ БЛОКА>
```
После на файловой системе станет доступна директория с файлами блока `/blocks/<ИМЯ БЛОКА>`.

# Шаблон
Начинайте разработку блока с шаблона. Что вам нужно от шаблона?

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

* Подробнее о `YBlock`: [ссылка](yblock.md)
* Подробнее об `inherit`: [ссылка](https://github.com/dfilatov/inherit)

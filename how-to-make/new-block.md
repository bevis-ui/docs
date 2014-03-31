в разработке


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
# Введите имя блока: input
```
После на файловой системе станет доступна директория с файлами блока `/blocks/input`.

# Шаблон
Начинайте разработку блока с шаблона. Что вам нужно от шаблона?

* Отображение в браузере в виде тега `input`.
* Возможность задать значение с помощью параметров.

Попробуем набросать первый вариант.
Открываем файл `input.bt.js` в директории `blocks/input`. Он выглядит так, это шаблон, заготовка.

```javascript
module.exports = function (bt) {
    bt.match('input', function (ctx) {
        ctx.setTag('span');

        ctx.setContent('Содержимое блока');
    });
};
```

Редактируем:

```javascript
module.exports = function (bt) {
    bt.match('input', function (ctx) {
        ctx.setTag('input');
        ctx.setAttr('value', ctx.getParam('value'));
    });
};
```

BT-шаблон - это обычный CommonJS (nodejs) модуль.

----

Показать page.js и место, где вызывается блок на странице.

----

Используем блок на странице:

```javascript
{
    block: 'input',
    value: 'Hello'
}
```

Отлично, все работает.


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

# Нужны разные отображения?

Если дизайнер для разных форм сайта нарисовал разные текстовые поля,
есть смысл сделать два  отдельных представления этого блока, два `view`. У них будут разные стили,
но одинаковое поведение. Придумаем имена для представлений: `normal` и `active`.


Сначала добавим поддержку `view` в шаблоны.

Одно из отображений можно сделать умолчательным, например `normal`, потому что именно оно используется
 на всех страницах сайта, а `active` только на одной странице. Умолчательным (по дефолту,
 если говорить сленговым языком) означает, что в декларации блока не нужно указывать `view: 'normal'`,
 это вью будет добавляться автоматически.

Добавим перед вызов метода `bt.setDefaultView('input', 'normal')` и в шаблонах к имени блока добавим маску `*`

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
    bt.setDefaultView('input', 'normal');

    bt.match('input*', function (ctx) {
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

// or

{
    block: 'input',
    view: 'active',
    value: 'Hello'
}

// или вообще без view: подставится дефолтное значение normal

{
    block: 'input',
    value: 'Hello'
}

```

Проверям: не работает. Потому что нет зависимостей.

* зависимости для вью

Проверяем: работает. В html выливаются правильные классы.

Теперь расширяем стили

* два файла styl
* селекторы с вью
* общшие стили выносим в миксины скинов.

Проверяем - работает.

----

* Как обращаться за данными и передавать в блок?


* Подробнее о `YBlock`: [ссылка](yblock.md)
* Подробнее об `inherit`: [ссылка](https://github.com/dfilatov/inherit)



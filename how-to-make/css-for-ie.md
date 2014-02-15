Islets поддерживает браузеры IE8+ в Standards Mode.

Сборка примеров документации (npm-пакет `"bevis-doc-builder": "0.0.3"`) настроена так, что собираются отдельно 3 вида стилей:

- `.ie8.css` — стили для IE8 и  ниже
- `.ie9.css` — стили для IE9
- `.css` — стили для нормальных браузеров

Таким образом через `conditional comments` в любом браузере отрабатывает только 1 файл стилей.

Для отделения стилей для IE от прочих мы используем переменную `ie` в значениях `8` и `9`.
Просто в stylus-файлах в нужных местах пишем `if ie == 8 {}`, `if ie != 8 {}` и т.д.

Примеры
```stylus
    if ie != 8 {
        &__box:after {
            transition: bottom .05s ease-out, opacity .05s ease-out;

            opacity: 0;
        }

        &._checked &__box:after {
            opacity: 1;
        }
    }
```

```stylus
    &:hover &__box {
        if ie == 8 { border-color: #b3b3b3; }
        if ie != 8 { border-color: rgba(0, 0, 0, 0.3); }
    }
```

title: Agenda
class: big

- Предыстория
- Karma. Основные понятия
- Как это работает
- Примеры
- Выводы

---

title: Предыстория
content_class: flexbox vcenter

Большой проект + Большая команда = Много кода

Появилась необходимость привести
в порядок большой участок кода(модуль).
Который несет ответственность за важную функциональность.

---

title: Инструменты
class: big

Какие инструменты были в моем распоряжении

- [Vim](http://www.vim.org) в качестве редактора
- [JsHint](http://jshint.com) для быстрой проверки синтаксиса
- [Mocha-Phantomjs](https://github.com/metaskills/mocha-phantomjs) для запуска тестов
- [Книжки по refactoring-у](http://www.amazon.com/gp/product/0201485672?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0201485672)
- Задача по улучшению тестов

И желание!

---

title: Проблемы
content_class: flexbox vcenter

Не все гладко было в этом королевстве.

- Много непонятного кода
- Нету тестов (для модуля)
- [Собственная система](https://github.com/bem/borschik) сборки зависимостей для JavaScript
- Make

---

title: Эволюция тестов в проекте
content_class: flexbox vcenter

- Нет тестов
- Запускаются в браузере
- Запускаются в браузере (Guard-livereload)
- Запускаются в терминале (mocha-phantomjs)
- Запускаются в терминале. Нет жесткой зависимы от окружения где выполняются тесты (Karma)*

*Еще в стадии внедрения в проект

---

title: Чего хотелось
content_class: flexbox vcenter

- Избавиться от файла index.html для тестов
- Использовать code coverage, code analyzer
- Интеграция с терминалом
- Возможность запускать тесты на CI сервере
- Nodejs/npm

---

title: Что предлагали
content_class: flexbox vcenter

- Grunt + PhantomJS + Make

Все тот же index.html + Make сборки файлов

<pre class="prettyprint" data-lang="mk">
$(NPM_BIN)/borschik -i dist/some-file.js --minimize=no -o dist/_some-processed-file.js
$(NPM_BIN)/grunt
$(MAKE) grunt-clean
</pre>

Проблема. Каждый раз надо добавлять файл в index.html и Make.
Чревато ошибкам.

---

title: Мысли
content_class: flexbox vcenter

## Karma?

...Вроде что-то, что может решить мои проблемы...

И задача есть под такое дело.

**Надо попробовать!**

---

title: Karma
content_class: flexbox vcenter

Karma - это инструмент для автоматического запуска JavaScript тестов, написанный на nodejs.

Не framework для написания тестов!

Автор Vojta Jína (Google, AngularJs Team member)

---

title: Основные особенности
content_class: flexbox vcenter

- Поддерживает большинство браузеров
- Обещает высокую скорость
- Надежный
- Расширяемый. Возможность использовать языки компилируемые в JavaScript
- Есть интеграция с IDE. Консольный интерфейс
- Интеграция с CI
- Простота

---

title: Установка
content_class: flexbox vcenter

Установка:
<pre class="prettyprint" data-lang="bash">
npm install karma
karma init
npm install mocha --save-dev
npm install karma-phantomjs-launcher --save-dev
npm install karma-expect
</pre>
Результат karma.conf.js

---

title: Настройка (karma.conf.js)

<pre class="prettyprint" data-lang="javascript">
module.exports = function(config) {
  config.set({
    basePath: '',
    frameworks: ['mocha', 'expect'],
    files: [
      'test/**/*_test.js'
    ],
    exclude: [],
    reporters: ['progress'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: true,
    browsers: ['PhantomJS'],
    captureTimeout: 60000,
    singleRun: false
});
</pre>

---
title: Files
content_class: flexbox vcenter

Избавляемся от index.html

<pre class="prettyprint" data-lang="javascript">
files: [

  // Простой шаблон для загрузки все файлов с тестами
  // Эквивалент для {pattern: 'test/unit/*.spec.js',
  // watched: true, served: true, included: true}
  'test/unit/*_test.js',

  // файл который загружается на сервер, но за которым не следим
  {pattern: 'compiled/index.html', watched: false},

  // файл за которым только следим
  {pattern: 'app/index.html', included: false, served: false}
  ],
]
</pre>

---
title: Plugins
content_class: flexbox vcenter

Frameworks:
<pre class="prettyprint" data-lang="javascript">
frameworks: ['mocha', 'expect'],
</pre>

Preprocessors:
<pre class="prettyprint" data-lang="javascript">
preprocessors: {
  '**/*.coffee': ['coffee'],
  '**/*.html': ['html2js']
},
</pre>

Reporters:
<pre class="prettyprint" data-lang="javascript">
reporters: ['progress', 'coverage'],
</pre>

---

title: Как это работает
content_class: flexbox vcenter

Клиент - Сервер

![Client - Server](images/karma-design.png)

Inversion of Control / Dependency Injection

---

title: На клиенте
content_class: flexbox vcenter

Клиент

![Client - Server](images/karma-implementation.png)

---

title: Примеры
content_class: flexbox vcenter

Репортер
<pre class="prettyprint" data-lang="javascript">
module.exports = function(config) {
  config.set({
    ...
    plugins:[
      {'reporter:console', ['type', function(){
        this.onBrowserComplete = function (browser) {
            console.log('Result: ', browser.lastResult.failed ? 'Fail' : 'Ok');
          }
      }]}
    ]
});
</pre>

---

title: karma-expect
content_class: flexbox vcenter

![karma-expect](images/karma-expect.png)
Пример framework-а

---

title: Borschik
content_class: flexbox vcenter

Загрузка зависимостей/Модульность в Yandex.Mail

<pre class="prettyprint" data-lang="javascript">
  /*borschik:include:common.js*/
</pre>

[borschik](https://github.com/bem/borschik)

---

title: Чуточку сложнее, но полезнее
content_class: flexbox vcenter

<pre class="prettyprint" data-lang="javascript">
var borschik = require('borschik');
var stream = require('./lib/stream');
var createBorschikPreprocessor = function(args, config, logger, helper) {
    config = config || {};
    var log = logger.create('preprocessor.borschik');
    var defaultOptions = {
        'comments': false,
        'freeze': false,
        'minimize': false,
        'tech': 'js'
    };
    var options = helper.merge(defaultOptions, args.options || {}, config.options || {});
    ...
};
createBorschikPreprocessor.$inject = ['args', 'config.borschikPreprocessor', 'logger', 'helper'];
module.exports = { 'preprocessor:borschik': ['factory', createBorschikPreprocessor] };
</pre>

---

title: ...
content_class: flexbox vcenter

<pre class="prettyprint" data-lang="javascript">
    return function(content, file, done) {
        var output = new stream.Writable();
        var result = null;
        var onError = function(errorObject) {
            log.error('%s\n  at %s', errorObject.message, file.originalPath);
        };
        log.debug('Processing "%s".', file.originalPath);
        var opts = helper._.clone(options);
        opts.input = file.originalPath;
        opts.output = output;
        output.on('data', function(data) {
            done(data);
        });
        try {
            result = borschik.api(opts).fail(onError);
        } catch (e) {
            onError(e);
            return;
        }
    };
</pre>

---

title: Выводы
content_class: flexbox vcenter

- Быстрый, надежный, гибкий
- Продуманная архитектура
- Много интересных решений
- Удобно!

---

title: Ссылки
content_class: flexbox vcenter

- [Сайт проекта](https://karma-runner.github.io)
- [Проект на github](https://github.com/karma-runner/karma)
- [Как это работает](https://github.com/karma-runner/karma/raw/master/thesis.pdf)
- [DI для Nodejs](https://github.com/vojtajina/node-di)
- [Коллекция мок-объектов для Nodejs](https://github.com/vojtajina/node-mocks)
- [Borschik preprocessor](https://github.com/maksimr/karma-borschik-preprocessor)

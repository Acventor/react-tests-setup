# Тестирование React приложений

Привет! Здесь собрана информация и примеры для удобной подготовки к тестированию React приложений.

- [Подготовка](#подготовка)
  - [Настройка](#настройка)
  - [Расположение и название тестовых файлов](#расположение-и-название-тестовых-файлов)
  - [Основной синтаксис тестов](#основной-синтаксис-тестов)
- [Вступление](#вступление)
  - [Что тестировать?](#что-тестировать)
  - [Что не тестировать?](#что-не-тестировать)
  - [Что использовать?](#что-использовать)
- [Частые вопросы](#частые-вопросы)
  - [Почему не стоит тестировать реализацию?](#почему-не-стоит-тестировать-реализацию)
  - [Как найти нужный DOM элемент?](#как-найти-нужный-dom-элемент)
  - [Как быть с состоянием/редьюсером компонента?](#как-быть-с-состоянием%2Fредьюсером-компонента)
  - [Что за ошибка `should be wrapped into act`?](#что-за-ошибка-should-be-wrapped-into-act)
  - [Как тестировать `fetch` запросы?](#как-тестировать-fetch-запросы)
- [Полезные материалы](#полезные-материалы)

## Подготовка

### Установка

Добавляем необходимые `npm пакеты`:

- jest
- nock
- @testing-library/jest-dom
- @testing-library/react
- @testing-library/user-event
- babel-jest
- eslint-plugin-jest
- eslint-plugin-jest-dom
- eslint-plugin-testing-library

После этого создаем и настраиваем файлы конфигурации `jest.config.js`, `jest.setup.js`, `.eslintrc.js`, `.babelrc`. Они уже есть в этом репозитории и их можно использовать в проектах в текущем или доработанном виде

### Запуск тестов

Вызов `npm run test` запускает одноразовую проверку тестов.

Вызов `npm run test:watch` запускает слежение за изменениями в файлах тестов и автоматически их выполняет.

### Расположение и название тестовых файлов

В проекте файлы с тестами располагаются либо в папке своего компонента, либо в папке `/tests` в корневой директории. Папка `/tests` содержит тесты страниц (`/pages`), функций-хелперов (`/helpers`) и другие.

Название тестового файла компонента:\
`ComponentName.test.js`

Название тестового файла функции:\
`functionName.test.js`

### Основной синтаксис тестов

Каждый тест оборачивается в функцию `it()` с описанием проверки:

```js
it('sends request on submit', () => {...})
```

Так же тесты можно группировать в смысловые блоки с помощью `describe()`:

```js
describe('Main logic tests', () => {
    it('does something', () => {...})
})
```

## Вступление

### Что тестировать?

В первую очередь, работоспособность компонентов и взаимодействие с приложением от лица юзера. Отрендерен ли компонент, нажатие на кнопки, заполнение форм, открытие модалок, подзагрузка элементов и др. Иными словами, результат нашего кода, а не содержание.

### Что не тестировать?

То, как компоненты или функции написаны и сделаны. Названия переменных, логика функций, типы props, стили, сторонние библиотеки и др.

Сторонние библиотеки (например, _Яндекс Карты_) имеют свои тесты. В наших мы можем либо полностью их заменить (например, используя заглушку типа _\</ div>_), либо тестировать только самую необходимую логику (например, отправку форм из _Formik_).

### Что использовать?

Для рендера и получения элементов используется библиотека [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro).

Для проверки элементов на содержимое и состояние используется [jest-dom](github.com/testing-library/jest-dom)

Для вызова событий из DOM используется [@testing-library/user-event](https://testing-library.com/docs/ecosystem-user-event/).

Для перехвата и подделывания `fetch` запросов используется [nock](github.com/nock/nock).

Синтаксис тестов и основные методы для работы с ними обеспечивает [nest](https://jestjs.io/docs/getting-started)

## Частые вопросы

### Почему не стоит тестировать реализацию?

Такой подход ведет к высокому шансу на то, что тест будет давать неверные результаты. Допустим, мы что-то незначительно поменяли в компоненте. В итоге остается полностью рабочий компонент, но устаревший тест будет говорить обратное (_False negatives_). Или наоборот, компонент может перестать работать, но тест будет думать, что проблем нет (_False positives_). И шансы на такие ошибки возрастают при малейшем рефакторе кода.

Поэтому мы тестируем _результат_, а не реализацию. Результат нашего кода, а не то, как он написан. Работаем именно с тем, что в конечно итоге дойдет до пользователя и будет непосредственно влиять на его опыт. По этой же причине мы не тестируем [снимками](https://jestjs.io/docs/snapshot-testing). И особенно приятно то, что такой подход не только надежнее, но и проще в реализации и поддержке.

Подробнее на эту тему можно почитать в [этом материале](https://kentcdodds.com/blog/testing-implementation-details).

### Как найти нужный DOM элемент?

Используем запросы `(queries)` от библиотеки [@testing-library/react](https://testing-library.com/docs/react-testing-library/cheatsheet#queries).

- Для элементов с определенной `aria` ролью используем `...ByRole` запросы.\
   Например:

  ```js
  screen.getByRole('button', { name: 'Some text' });
  ```

  В этом примере мы, во-первых, будем уверены в том, что найденный элемент является именно кнопкой, а во-вторых, проверяем ее же доступность. Список [ARIA Roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles).

- Для разных элементов есть более подходящие `queries`, чем другие. Например, для полей форм лучше использовать `getByLabelText`.
- Различия между `getBy...`, `queryBy...` и `findBy...`:
  - `getBy...` находит элемент и выбрасывает ошибку, если он не найден.
  - `queryBy...` находит элемент и возвращает `null`, если он не найден (удобно для проверки на отсутствие).
  - `findBy...` возвращает выполненный промис, если элемент был найден за определенное время (удобно в случае, когда элемент рендерится не сразу).
- Для получения сразу нескольких элементов используем `getByAll...`, `queryByAll...` и `findByAll...`.

### Как быть с состоянием/редьюсером компонента?

Никак не используем оригинальные `state` или `reducer`, а проверяем и оцениваем только результат их работы. Например то, что при клике на кнопку должна появиться модалка.

Примеры есть в `/examples/component-with-state` и `/examples/component-with-reducer`.

### Что за ошибка `should be wrapped into act`?

Такая ошибка часто возникает из-за асинхронных действий в компоненте. Тест по умолчанию работает в синхронном режиме и если после его завершения останутся не закрытые асинхронные действия, он выбросит эту ошибку. Её можно часто встретить при тестах форм под управлением `Formik` или при вызове асинхронных колбеков. Есть несколько решений данной проблемы:

- Обернуть действие в `await waitFor()`. Но так лучше [не делать](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#performing-side-effects-in-waitfor), потому что `waitFor()` нужен только для проверки _после_ асинхронных действий. Например, очистились ли поля формы после завершения асинхронного `onSubmit`? Такой метод наиболее предпочтителен.
- Обернуть асинхронное действие в `act()`. Но такой способ используется [в очень редких](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning) случаях.
- Поставить вызов `await waitFor(() => {})` с пустым колбеком в конец теста. Это заставит тест дождаться завершения всех асинхронных действий перед закрытием. Хоть это и своего рода хак, но его можно использовать, если нет варианта получше.

### Как тестировать `fetch` запросы?

С помощью HTTP перехватчика [nock](github.com/nock/nock). Эта библиотека перехватывает настоящие запросы и возвращает любой желаемый ответ. Она удобна тем, что нам не нужно задумываться, как мокать функцию для вызова запроса, ведь можно работать с ним напрямую.

Например:
```
nock(BASE_API_URL).get(API_PATH).reply(200, { some_data: [] });
```

Примеры использования `nock` в тестах:\
`/examples/component-with-form`

## Полезные материалы

В этих материалах могут встречаться другие библиотеки или методы тестирования. Поэтому этот список скорее нужен для ознакомления с правильными подходами к написанию тестов, а за примерами иногда лучше заглянуть в `/examples`.

- [Введение в тестирование](https://www.freecodecamp.org/news/testing-react-hooks/) (есть устаревшее)
- [Современные методы тестирования](https://blog.sapegin.me/all/react-testing-3-jest-and-react-testing-library/)
- [Распространенные ошибки и их решения](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [Почему нужно тестировать результат, а не реализацию](https://kentcdodds.com/blog/testing-implementation-details)
- [Список `expect` методов библиотеки Jest](https://jestjs.io/docs/expect)
- [Список сопоставителей `(matchers)` библиотеки jest-dom](https://github.com/testing-library/jest-dom)

Так же советую почитать статьи от одного из лучших мастеров тестирования – [Kent C. Dodds](https://kentcdodds.com/blog/?q=testing).

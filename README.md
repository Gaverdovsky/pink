# Стартовый проект с gulp

ВНИМАНИЕ! Этот проект в разработке (призван систематизировать мои личные наработки из области вёрстки), я добавляю и меняю в нем довольно много всего. Это даже не альфа. Периодически может выглядеть странно и не менее странно работать.

Установка: выполнить в папке проекта: `npm i`.

Использование: `npm start` (дабы не ставить глобально 4-ю версию gulp, которая сейчас в альфе). После `npm start` можно через пробел указать задачу. К примеру `npm start less` выполнит только задачу `less` из `gulpfile.js` (полный список задач смотри в `gulpfile.js`).

`port=3004 npm start` — для запуска сервера автообновлений на указанном порту.

`NODE_ENV=production npm start [задача]` — для запуска задач без создания sourcemaps.



## Парадигма

- Используется именование классов и файлов по БЭМ.
- Каждый БЭМ-блок в своей папке внутри `/src/blocks/` (less, js, картинки, разметка; обязателен только less-файл).
- Есть глобальные файлы: css, js, шрифты, картинки, less-файлы (переменные, глобальная стилизация...).
- Есть диспетчер подключений `/src/less/style.less`. Если в нем импортирован less-файл какого-либо блока, этот блок считается используемым (обрабатывается его js и доп. файлы).
- Для разметки можно использовать [микрошаблонизацию](https://www.npmjs.com/package/gulp-file-include) — обычное включение разметки мелких файлов внутрь больших файлов с возможностью передавать какие-то данные в мелкие файлы. А можно и не использовать.



### Блоки

Каждый блок лежит в `/src/blocks/` в своей папке. Каждый блок — как минимум, папка и одноимённый less-файл.

Возможное содержимое блока:

```bash
block-name/               # Папка блока
  img/                    # Изображения, используемые блоком и обрабатываемые автоматикой сборки
  img_to_bg/              # Изображения, не обрабатываемые автоматикой сборки
  demo-block.less         # Главный стилевой файл блока
  demo-block--mod.less    # Отдельный файл модификатора блока (если объемный и нужен не на всех проектах)
  demo-block.js           # Главный js-файл блока
  demo-block--mod.js      # js-файл для отдельного модификатора блока
  demo-block.html         # Варианты разметки (как документация блока или как вставляемый микрошаблонизатором фрагмент)
  demo-block.css          # Добавочный css (копируется как отдельный файл в `build/css`)
  library.js              # Дополнительная js-библиотека, берется в конкатенируемый js-файл проекта (имя любое, в конкатенацию берется перед всеми прочими файлами, повторения ищутся по имени файла)
  readme.md               # Какое-то пояснение
```

### Удобное создание нового блока


```bash
# формат: node createBlock.js [имя блока] [доп. расширения через пробел]
node createBlock.js block # создаст только папку, block.html и block.less
node createBlock.js new-block js jade # создаст папку, new-block.html, new-block.less, new-block.js, new-block.jade
```

По умолчанию будут созданы `.html` и `.less` файлы, в них будет записан стартовый контент.

Если блок уже существует, файлы не будут затёрты, но создадутся те файлы, которые ещё не существуют.

После создания блока, в диспетчер подключений будет дописана (в самый низ) строка импорта стилевого файла.



## Подключение блоков

Если в диспетчере подключений (`./src/less/style.less`):

```
@import './src/blocks/demo-block/demo-block.less';
```

То указанный файл будет взят в компиляцию стилей, а так же:
- в обработку будет взят js-файл блока: `./src/blocks/demo-block/demo-block.js` (если существует и не пустой, будет конкатенирован с прочими js-файлами и углифицирован)
- в обработку будет взят css-файл блока: `./src/blocks/demo-block/demo-block.css` (если существует и не пустой, будет скопирован в папку сборки и минифицирован)
- в обработку будут добавлены все картинки блока: `./src/blocks/demo-block/img/*.{jpg,jpeg,gif,png,svg}` (если в папке блока существует подпапка `img/`)

Если в диспетчере подключений:

```
@import './src/blocks/demo-block/demo-block.less';
@import './src/blocks/demo-block/demo-block--mod.less';
```

То указанные файлы будет взяты в компиляцию стилей, а так же:
- в обработку будет взят js-файл блока: `./src/blocks/demo-block/demo-block.js` (если существует и не пустой, будет конкатенирован с прочими js-файлами и углифицирован)
- в обработку будет взят js-файл блока: `./src/blocks/demo-block/demo-block--mod.js` (если существует и не пустой, будет конкатенирован с прочими js-файлами и углифицирован)
- в обработку будет взят css-файл блока: `./src/blocks/demo-block/demo-block.css` (если существует и не пустой, будет скопирован в папку сборки и минифицирован)
- в обработку будет взят css-файл блока: `./src/blocks/demo-block/demo-block--mod.css` (если существует и не пустой, будет скопирован в папку сборки и минифицирован)
- в обработку будут добавлены все картинки блока: `./src/blocks/demo-block/img/*.{jpg,jpeg,gif,png,svg}` (если в папке блока существует подпапка `img/`)



## Назначение папок

```bash
build/          # Сюда собирается проект, здесь работает сервер автообновлений.
src/            # Исходные файлы
  _include/     # - фрагменты html для самого верха (секция head) и самого низа (перед закрывающим body) страницы
  blocks/       # - блоки (компоненты) проекта
  css/          # - глобальный css-файл (будет скопирован только если существует и не пустой)
  fonts/        # - шрифты проекта (никак не обрабатываются, см. http://jaicab.com/localFont/)
  img/          # - глобальные картинки (будут обработаны только из корня этой папки, подпапки игнорируются)
  js/           # - глобальный js-файл (обработается только если существует и не пустой), фреймворки (только копируются, могут быть подключены вручную)
  less/         # - диспетчер подключений и глобальные стили
  blocks_library.html   # библиотека блоков проекта
  index.html            # главная страница проекта
```



## Комментирование для разработчиков

Для разметочных файлов можно использовать комментарии вида `<!--DEV Комментарий -->` — такие комментарии не попадут в собранный html.



## Выгрузка на gh-pages

Содержимое проекта можно быстро «выгрузить» на [gh-pages](https://help.github.com/articles/user-organization-and-project-pages/#project-pages) (автоматически запушить содержимое папки `build/` в ветку `gh-pages` репозитория проекта). Адрес для просмотра будет таким: http://USERNAME.github.io/PROJECTNAME/ (полное повторений файловой структуры папки `build/`).


```bash
npm run deploy    # собрать проект без карт кода и отправить на gh-pages
```

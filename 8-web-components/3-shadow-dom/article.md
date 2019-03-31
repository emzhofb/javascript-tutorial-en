# Shadow DOM

Did you ever think how complex browser controls like `<input type="range">` are created and styled?

The browser uses DOM/CSS internally to draw this kind of thing:

<input type="range">

The associated DOM structure is normally hidden from us, but we can see it in developer tools. E.g. in Chrome, we need to enable in Dev Tools "Show user agent shadow DOM" option.

Then `<input type="range">` looks like this:

![](shadow-dom-range.png)

What you see under `#shadow-root` is called "Shadow DOM".

We can't get built-in Shadow DOM elements by regular JavaScript calls or selectors. These are not regular children, but a powerful encapsulation technique.

In the example above, we can see a useful attribute `pseudo`. It's non-standard, exists for historical reasons. We can use it style subelements with CSS, like this:

```html run autorun
<style>
/* make the slider track red */
input::-webkit-slider-runnable-track {
  background: red;
}
</style>

<input type="range">
```

Once again, `pseudo` is a non-standard attribute. Chronologically, browsers first started to experiment with internal DOM structures to implement controls, and then, after time, Shadow DOM was standartized to allow us, developers, to do the similar thing.

Furhter on, we'll use the modern Shadow DOM standard.

## Shadow tree

A DOM element can have two types of DOM subtrees:

1. Light tree -- a regular DOM subtree, made of HTML children. All subtrees that we've seen before are light.
2. Shadow tree -- a hidden DOM subtree, not reflected in HTML, hidden from prying eyes.

If an element has both at the same time, then the browser renders only the shadow tree. But we can setup a kind of composition between two trees as well. We'll see the details later in the chapter <info:slots-composition>.

Shadow tree can be used in Custom Elements to hide internal implementation details.

For example, this `<show-hello>` element hides its internal DOM in shadow tree:

```html run autorun height=40
<script>
customElements.define('show-hello', class extends HTMLElement {
  connectedCallback() {
    const shadow = this.attachShadow({mode: 'open'});
    shadow.innerHTML = `<p>Hello, ${this.getAttribute('name')}!</p>`;
  }  
});
</script>

<show-hello name="John"></show-hello>
```

The call to `elem.attachShadow({mode: …})` creates a shadow tree root for the element. There are two limitations:
1. We can create only one shadow root per element.
2. The `elem` must be either a custom element, or one of: "article", "aside", "blockquote", "body", "div", "footer", "h1..h6", "header", "main" "nav", "p", "section", "span".

The `mode` option sets the encapsulation level. It must have any of two values:
- **`"open"`** -- then the shadow root is available as DOM property `elem.shadowRoot`, so any code is able to access it.   
- **`"closed"`** -- `elem.shadowRoot` is always `null`. Browser-native shadow trees, such as  `<input type="range">`, are closed.

The [shadow root object](https://dom.spec.whatwg.org/#shadowroot) is like an element: we can use `innerHTML` or DOM methods to populate it.

That's how the resulting DOM looks in Chrome dev tools:

![](shadow-dom-say-hello.png)

Shadow DOM is strongly delimited from the main document:

1. Shadow DOM elements are not visible to `querySelector` from the light DOM. In particular,  Shadow DOM elements may have ids that conflict with those in the light DOM. They be unique only within the shadow tree.
2. Shadow DOM has own stylesheets. Style rules from the outer DOM don't get applied.

For example:

```html run untrusted height=40
<style>
*!*
  /* document style doesn't apply to shadow tree (2) */
*/!*
  p { color: red; }
</style>

<div id="elem"></div>

<script>
  elem.attachShadow({mode: 'open'});
*!*
    // shadow tree has its own style (1)
*/!*
  elem.shadowRoot.innerHTML = `
    <style> p { font-weight: bold; } </style>
    <p>Hello, John!</p>
  `;

*!*
  // <p> is only visible from queries inside the shadow tree (3)
*/!*
  alert(document.querySelectorAll('p').length); // 0
  alert(elem.shadowRoot.querySelectorAll('p').length); // 1
</script>  
```

1. The style from the document does not affect the shadow tree.
2. ...But the style from the inside works.
3. To get elements in shadow tree, we must query from inside the tree.

The element with a shadow root is called a "shadow tree host", and is available as the shadow root `host` property:

```js
// assuming {mode: "open"}, otherwise elem.shadowRoot is null
alert(elem.shadowRoot.host === elem); // true
```


## Slots, composition

Quite often, our components should be generic. Like tabs, menus, etc -- we should be able to fill them with content.

Something like this:


<custom-menu>
  <title>Please choose:</title>
  <item>Candy</item>
  <item>
  <div slot="birthday">01.01.2001</div>
  <p>I am John</p>
</user-card>


Let's say we need to create a `<user-card>` custom element, for showing user cards.

```html run
<user-card>
  <p>Hello</p>
  <div slot="username">John Smith</div>
  <div slot="birthday">01.01.2001</div>
  <p>I am John</p>
</user-card>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot = `
      <slot name="username"></slot>
      <slot name="birthday"></slot>
      <slot></slot>
    `;
  }
});
</script>
```




Let's say we'd like to create a generic `<user-card>` element for showing the information about the user. The element itself should be generic and contain "bio", "birthday" and "avatar" image.

 Another developer should be able to add `<user-card>` on the page


Shadow DOM with "slots",  e.g. articles, tabs, menus, that can be filled by

polyfill https://github.com/webcomponents/webcomponentsjs








<hr>

[Shadow tree](https://dom.spec.whatwg.org/#concept-shadow-tree) is a DOM tree that отдельным стандартом. Частично он уже используется для обычных DOM-элементов, но также применяется для создания веб-компонентов.

*Shadow DOM* -- это внутренний DOM элемента, который существует отдельно от внешнего документа. В нём могут быть свои ID, свои стили и так далее. Причём снаружи его, без применения специальных техник, не видно, поэтому не возникает конфликтов.

## Внутри браузера

Концепция Shadow DOM начала применяться довольно давно внутри самих браузеров. Когда браузер показывает сложные элементы управления, наподобие слайдера `<input type="range">` или календаря `<input type="date">` -- внутри себя он конструирует их из самых обычных стилизованных `<div>`, `<span>` и так далее.

С первого взгляда они незаметны, но если в настройках Chrome Development Tools выбрать показ Shadow DOM, то их можно легко увидеть.

Например, вот такое содержимое будет у `<input type="date">`:
![](shadow-dom-chrome.png)

То, что находится под `#shadow-root` -- это и есть Shadow DOM.

**Получить элементы из Shadow DOM можно только при помощи специальных JavaScript-вызовов или селекторов. Это не обычные дети, а намного более мощное средство отделения содержимого.**

В Shadow DOM выше можно увидеть полезный атрибут `pseudo`. Он нестандартный, существует по историческим причинам. С его помощью можно стилизовать подэлементы через CSS, например, сделаем поле редактирования даты красным:

```html run no-beautify
<style>
*!*
input::-webkit-datetime-edit {
*/!*
  background: red;
}
</style>

<input type="date">
```

Ещё раз заметим, что `pseudo` -- нестандартный атрибут. Если говорить хронологически, то сначала браузеры начали экспериментировать внутри себя с инкапсуляцией внутренних DOM-структур, а уже потом, через некоторое время, появился стандарт Shadow DOM, который позволяет делать то же самое разработчикам.

Далее мы рассмотрим работу с Shadow DOM из JavaScript, по стандарту [Shadow DOM](http://w3c.github.io/webcomponents/spec/shadow/).

## Создание Shadow DOM

Shadow DOM можно создать внутри любого элемента вызовом `elem.createShadowRoot()`.

Например:

```html run autorun="no-epub"
<p id="elem">Доброе утро, страна!</p>

<script>
  var root = elem.createShadowRoot();
  root.innerHTML = "<p>Привет из подполья!</p>";
</script>
```

Если вы запустите этот пример, то увидите, что изначальное содержимое элемента куда-то исчезло и показывается только "Привет из подполья!". Это потому, что у элемента есть Shadow DOM.

**С момента создания Shadow DOM обычное содержимое (дети) элемента не отображается, а показывается только Shadow DOM.**

Внутрь этого Shadow DOM, при желании, можно поместить обычное содержимое. Для этого нужно указать, куда. В Shadow DOM это делается через "точку вставки" (insertion point). Она объявляется при помощи тега `<content>`, например:

```html run autorun="no-epub"
<p id="elem">Доброе утро, страна!</p>

<script>
  var root = elem.createShadowRoot();
  root.innerHTML = "<h3>*!*<content></content>*/!*</h3> <p>Привет из подполья!</p>";
</script>
```

Теперь вы увидите две строчки: "Доброе утро, страна!" в заголовке, а затем "Привет из подполья".

Shadow DOM примера выше в инструментах разработки:

![](shadow-content.png)

Важные детали:

- Тег `<content>` влияет только на отображение, он не перемещает узлы физически. Как видно из картинки выше, текстовый узел  "Доброе утро, страна!" остался внутри `p#elem`. Его можно даже получить при помощи `elem.firstElementChild`.
- Внутри `<content>` показывается не элемент целиком `<p id="elem">`, а его содержимое, то есть в данном случае текст "Доброе утро, страна!".

**В `<content>` атрибутом `select` можно указать конкретный селектор содержимого, которое нужно переносить. Например, `<content select="h3"></content>` перенесёт только заголовки.**

Внутри Shadow DOM можно использовать `<content>` много раз с разными значениями `select`, указывая таким образом, где конкретно какие части исходного содержимого разместить. Но при этом дублирование узлов невозможно. Если узел показан в одном `<content>`, то в следующем он будет пропущен.

Например, если сначала идёт `<content select="h3.title">`, а затем `<content select="h3">`, то в первом `<content>` будут показаны заголовки `<h3>` с классом `title`, а во втором -- все остальные, кроме уже показанных.</li>

В примере выше тег `<content></content>` внутри пуст. Если в нём указать содержимое, то оно будет показано только в том случае, если узлов для вставки нет. Например потому что ни один узел не подпал под указанный `select`, или все они уже отображены другими, более ранними `<content>`.

Например:

```html run autorun="no-epub" no-beautify
<section id="elem">
  <h1>Новости</h1>
  <article>Жили-были <i>старик со старухой</i>, но недавно...</article>
</section>

<script>
  var root = elem.createShadowRoot();

  root.innerHTML = "<content select='h1'></content> \
   <content select='.author'>Без автора.</content> \
   <content></content>";

</script>

<button onclick="alert(root.innerHTML)">root.innerHTML</button>
```

При запуске мы увидим, что:

- Первый `<content select='h1'>` выведет заголовок.
- Второй `<content select=".author">` вывел бы автора, но так как такого элемента нет -- выводится содержимое самого `<content select=".author">`, то есть "Без автора".
- Третий `<content>` выведет остальное содержимое исходного элемента -- уже без заголовка `<h1>`, он выведен ранее!

Ещё раз обратим внимание, что `<content>` физически не перемещает узлы по DOM. Он только показывает, где их отображать, а также, как мы увидим далее, влияет на применение стилей.

## Корень shadowRoot

После создания корень внутреннего DOM-дерева доступен как `elem.shadowRoot`.

Он представляет собой специальный объект, поддерживающий основные методы CSS-запросов и подробно описанный в стандарте как [ShadowRoot](http://w3c.github.io/webcomponents/spec/shadow/#shadowroot-object).

Если нужно работать с содержимым в Shadow DOM, то нужно перейти к нему через `elem.shadowRoot`. Можно и создать новое Shadow DOM-дерево из JavaScript, например:

```html run autorun="no-epub"
<p id="elem">Доброе утро, страна!</p>

<script>
*!*
  // создать новое дерево Shadow DOM для elem
*/!*
  var root = elem.createShadowRoot();

  root.innerHTML = "<h3><content></content></h3> <p>Привет из подполья!</p> <hr>";
</script>

<script>
*!*
  // прочитать данные из Shadow DOM для elem
*/!*
  var root = elem.shadowRoot;
  // Привет из подполья!
  document.write("<p>p:" + root.querySelector('p').innerHTML);
  // пусто, так как физически узлы - вне content
  document.write("<p>content:" + root.querySelector('content').innerHTML);
</script>
```

```warn header="Внутрь встроенных элементов так \"залезть\" нельзя"
На момент написания статьи `shadowRoot` можно получить только для Shadow DOM, созданного описанным выше способом, но не встроенного, как в элементах типа `<input type="date">`.
```

## Итого

Shadow DOM -- это средство для создания отдельного DOM-дерева внутри элемента, которое не видно снаружи без применения специальных методов.

- Ряд браузерных элементов со сложной структурой уже имеют Shadow DOM.
- Можно создать Shadow DOM внутри любого элемента вызовом `elem.createShadowRoot()`. В дальнейшем его корень будет доступен как `elem.shadowRoot`. У встроенных элементов он недоступен.
- Как только у элемента появляется Shadow DOM, его изначальное содержимое скрывается. Теперь показывается только Shadow DOM, который может указать, какое содержимое хозяина куда вставлять, при помощи элемента `<content>`. Можно указать селектор `<content select="селектор">` и размещать разное содержимое в разных местах Shadow DOM.
- Элемент `<content>` перемещает содержимое исходного элемента в Shadow DOM только визуально, в структуре DOM оно остаётся на тех же местах.

Подробнее спецификация описана по адресу <http://w3c.github.io/webcomponents/spec/shadow/>.

Далее мы рассмотрим работу с шаблонами, которые также являются частью платформы Web Components и не заменяют существующие шаблонные системы, но дополняют их важными встроенными в браузер возможностями.
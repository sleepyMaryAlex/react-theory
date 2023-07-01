# ?Web components

_Веб-компоненты_ — это повторно используемые клиентские компоненты, основанные на официальных веб-стандартах и ​​поддерживаемые всеми основными браузерами.

Это отличный способ инкапсулировать функциональность из остального кода. Мало того, вы можете повторно использовать их в каждом веб-приложении и веб-странице.

Как разработчик, вы можете использовать React в своих веб-компонентах, или использовать веб-компоненты в React, или и то, и другое.

Основное отличие react-компонентов от веб-компонентов заключается в том, что мы можем использовать компоненты React только внутри приложения React. С другой стороны, веб-компонент можно использовать где угодно. Мы можем использовать его в React, Vue, Angular или любом другом веб-приложении.

Веб-компоненты состоят из трех основных технологий:

1. Custom elements
2. HTML templates
3. Shadow DOM

### Пример пользовательского элемента. Веб компонент в реакт

Давайте продолжим и создадим наш первый веб-компонент, который мы интегрируем в приложение React.

Вот код для нашего пользовательского элемента `hello-world`:

* Создайте класс, который расширяет `HTMLElement`, используя синтаксис класса.
* Зарегистрируйте элемент, используя метод `window.customElements.define()`.

~~~
class HelloWorldComponent extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<h1>Hello world!</h1>`;
  }
}

customElements.define("hello-world", HelloWorldComponent);

export default HelloWorldComponent;
~~~

Здесь мы используем веб компонент в реакт. Мы можем легко включить наш компонент следующим образом:

~~~
import "./HelloWorldComponent";

function App() {
  return (
    <div>
      <hello-world></hello-world>
    </div>
  );
}

export default App;
~~~

Тем не менее, в редакторе есть ошибка: `«Свойство «привет-мир» не существует для типа «JSX.IntrinsicElements»»`. Ошибка возникает из-за того, что компоненты React всегда должны начинаться с заглавной буквы. Проблема в том, если мы исправим на заглавную букву, браузер выдаст исключение, потому что мы нарушим одно из двух правил, которым необходимо следовать для имени тега элемента:

1. Имя должно начинаться со строчной буквы ASCII. Нигде не должно быть символов ASCII в верхнем регистре.
2. Имя должно содержать дефис  — символ. Этот символ тире помогает браузеру отличать пользовательские элементы от обычных.

Итак, как исправить ошибку? Typescript дает нам возможность объявлять собственные типы. Чтобы исправить ошибку, мы можем определить наш пользовательский тип элемента, расширив `IntrinsicElements` интерфейс React следующим образом:

types.d.ts
~~~
export declare global {
  namespace JSX {
    interface IntrinsicElements {
      "hello-world": React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >;
    }
  }
}
~~~

Чтобы получить доступ к необходимому API веб-компонентов, необходимо использовать реф для взаимодействия с DOM-узлом напрямую. Если вы используете сторонние веб-компоненты, лучшим решением будет создать React-компонент и использовать его как обёртку для веб-компонента.

События, созданные веб-компонентами, могут неправильно распространяться через дерево React-компонентов. Вам нужно вручную добавить обработчики для таких событий в собственные React-компоненты.

Веб-компоненты используют `«class»` вместо `«className»`.

### Пользовательский элемент. React в веб компоненте

Можно использовать реакт в веб-компонентах:

~~~
class XSearch extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement("span");
    this.attachShadow({ mode: "open" }).appendChild(mountPoint);

    const name = this.getAttribute("name");
    const url =
      "https://www.google.com/search?q=" + encodeURIComponent(name as string);
    const root = ReactDOM.createRoot(mountPoint);
    root.render(<a href={url}>{name}</a>);
  }
}

customElements.define("x-search", XSearch);
~~~

### Пример Shadow DOM

* Используется для автономных компонентов.
* Содержит стиль и разметку компонента.
* Создает `«shadowRoot»`, на который можно ссылаться позже, когда это необходимо.
* Прикрепляется к элементу с помощью `element.attachShadow({ mode: "open" })`.
* Метод `document.querySelector()` не работает для узлов в ShadowDOM.

В этом примере мы создали теневой DOM и прикрепили его к нашему пользовательскому элементу.

App.tsx
~~~
import "./WebComponent";

function App() {
  return <test-element></test-element>;
}
~~~

types.d.ts
~~~
export declare global {
  namespace JSX {
    interface IntrinsicElements {
      "test-element": React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >;
    }
  }
}
~~~

WebComponent.ts
~~~
class TestElement extends HTMLElement {
  connectedCallback() {
    const shadowRoot = this.attachShadow({ mode: "open" });
    shadowRoot.innerHTML = `
    <style>
      div {
        background:cyan;
      }
    </style>
    <div>
        <p>Hello Mary!</p>
        <p>Nice to meet you</p>
    </div>`;
  }
}

customElements.define("test-element", TestElement);

export default TestElement;
~~~

### Пример HTML-templates

* Шаблоны могут содержать как HTML, так и CSS.
* Шаблоны могут быть определены в документе с помощью тега.
* Они используются для определения структур разметки.
* Элемент не отображается на странице, пока мы не прикрепим его к DOM с помощью JavaScript.

Простой пример:

App.tsx
~~~
import "./WebComponent";

function App() {
  return (
    <>
      <div>Hello!</div>
      <search-result></search-result>
    </>
  );
}

export default App;
~~~

types.d.ts
~~~
export declare global {
  namespace JSX {
    interface IntrinsicElements {
      "search-result": React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >;
    }
  }
}
~~~

WebComponent.ts
~~~
const template = document.createElement('template');
template.innerHTML = `
  <div>
    <p>The Google search result of your name is <a target="_blank" rel="noopener">here</a></p>
  </div>
`;

class SearchResult extends HTMLElement {
  connectedCallback() {
    const shadowRoot = this.attachShadow({ mode: 'open' });
    shadowRoot.appendChild(template.content.cloneNode(true));
    console.log(shadowRoot.querySelector('a') as HTMLAnchorElement);
    (shadowRoot.querySelector('a') as HTMLAnchorElement).href = '';
  }
}

customElements.define('search-result', SearchResult);

export default SearchResult;
~~~

В этом примере мы создали собственный шаблон для отображения данных учащихся:

App.tsx
~~~
import "./WebComponent";

function App() {
  return (
    <>
      <ul id="students"></ul>
      <test-element></test-element>
    </>
  );
}

export default App;
~~~

types.d.ts
~~~
export declare global {
  namespace JSX {
    interface IntrinsicElements {
      "test-element": React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >;
    }
  }
}
~~~

WebComponent.ts
~~~
const template = document.createElement("template");
template.innerHTML = `
  <li>
    <span class="name"></span> —
    <span class="score"></span>
  </li>
`;

class TestElement extends HTMLElement {
  connectedCallback() {
    const shadowRoot = this.attachShadow({ mode: "open" });
    const students = [
      { name: "Emma", score: 94 },
      { name: "Ashton", score: 72 },
    ];

    students.forEach((student) => {
      shadowRoot.appendChild((template as HTMLTemplateElement).content.cloneNode(true));

      (shadowRoot.querySelector(".name") as HTMLElement).innerHTML =
        student.name;
      (shadowRoot.querySelector(".score") as HTMLElement).innerHTML = String(student.score);

      document.getElementById("students")?.appendChild(shadowRoot);
    });
  }
}

customElements.define("test-element", TestElement);

export default TestElement;
~~~

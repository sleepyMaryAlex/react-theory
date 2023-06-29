# ?Portals

_Порталы_ позволяют рендерить дочерние элементы в DOM-узел, который находится вне DOM-иерархии родительского компонента.

`createPortal(child, container)`

* Первый аргумент (`child`) — это любой React-компонент, который может быть отрендерен, такой как элемент, строка или фрагмент.
* Следующий аргумент (`container`) — это DOM-элемент.

Обычно, когда вы возвращаете элемент из рендер-метода компонента, он монтируется в DOM как дочерний элемент ближайшего родительского узла. Но иногда требуется поместить потомка в другое место в DOM. 

Типовой случай применения порталов — когда в родительском компоненте заданы стили `overflow: hidden` или `z-index`, но вам нужно чтобы дочерний элемент визуально выходил за рамки своего контейнера. Например, диалоги, всплывающие карточки и всплывающие подсказки.

Портал может находиться в любом месте DOM-дерева. Несмотря на это, во всех других аспектах он ведёт себя как обычный React-компонент.

Такие возможности, как контекст, работают привычным образом, даже если потомок является порталом, поскольку сам портал всё ещё находится в React-дереве, несмотря на его расположение в DOM-дереве.

Так же работает и всплытие событий. Событие, сгенерированное изнутри портала, будет распространяться к родителям в содержащем React-дереве, даже если эти элементы не являются родительскими в DOM-дереве.

Представим следующую HTML-структуру:

~~~
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
~~~

Родительский компонент в `#app-root` сможет поймать неперехваченное всплывающее событие из соседнего узла `#modal-root`:

index.tsx
~~~
const root = ReactDOM.createRoot(
  document.getElementById("app-root") as HTMLElement
);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
~~~

App.tsx
~~~
function App() {
  const [clicks, setClicks] = useState(0);

  function handleClick() {
    // Эта функция будет вызвана при клике на кнопку в компоненте Child,
    // обновляя состояние компонента Parent, несмотря на то,
    // что кнопка не является прямым потомком в DOM.
    setClicks(clicks + 1);
  }

  return (
    <div onClick={handleClick} className="parent">
      <p>Количество кликов: {clicks}</p>
      <p>
        Откройте DevTools браузера, чтобы убедиться, что кнопка не является
        потомком блока div c обработчиком onClick.
      </p>
      <Modal>
        <Child />
      </Modal>
    </div>
  );
}
~~~

Modal.tsx
~~~
const modalRoot = document.getElementById("modal-root");

function Modal({ children }: { children: JSX.Element }) {
  const el = document.createElement("div");

  useEffect(() => {
    // Элемент портала добавляется в DOM-дерево после того, как
    // потомки компонента Modal будут смонтированы, это значит,
    // что потомки будут монтироваться на неприсоединённом DOM-узле.
    // Если дочерний компонент должен быть присоединён к DOM-дереву
    // сразу при подключении, например, для замеров DOM-узла,
    // или вызова в потомке 'autoFocus', добавьте в компонент Modal
    // состояние и рендерите потомков только тогда, когда
    // компонент Modal уже вставлен в DOM-дерево.
    (modalRoot as HTMLElement).appendChild(el);
    return () => {
      (modalRoot as HTMLElement).removeChild(el);
    };
  });

  return createPortal(children, el);
}
~~~

Child.tsx
~~~
function Child() {
  // Событие клика на этой кнопке будет всплывать вверх к родителю,
  // так как здесь не определён атрибут 'onClick'
  return (
    <div className="modal">
      <button>Кликните</button>
    </div>
  );
}
~~~

### Как исправить без порталов

Первая проблема с `overflow: hidden` у родительского элемента.

~~~
.parent {
  overflow: hidden;
  background-color: antiquewhite;
  width: 200px;
  height: 200px;
}

.modal {
  background-color: aqua;
  width: 100px;
  height: 100px;
}
~~~

Можно исправить без порталов, указав `position: fixed` или `absolute` у дочернего элемента, но родительский не должен быть `position: relative`.

Вторая проблема с `z-index`, что дочерний элемент ограничен контекстом стека его родителя.

~~~
.parent {
  background-color: antiquewhite;
  width: 200px;
  height: 200px;
  position: absolute;
  z-index: 3;
}

.modal {
  background-color: aqua;
  width: 100px;
  height: 100px;
  position: absolute;
  top: 170px;
  z-index: -1;
}
~~~

Проблему можно исправить и без портала. Например, переместить модал за пределы родительского элемента контента в основной контекст стека страницы или удалить позиционирование и `z-index` для родительского элемента, чтобы оно не ограничивало `z-index` модала.

# ?useRef

`useRef` возвращает изменяемый `ref`-объект, свойство `.current` которого инициализируется переданным аргументом (`initialValue`). Возвращённый объект будет сохраняться в течение всего времени жизни компонента.

`useRef()` создаёт обычный JavaScript-объект. Единственная разница между `useRef()` и просто созданием самого объекта `{current: ...}` — это то, что хук `useRef` даст один и тот же объект с рефом при каждом рендере.
Имейте в виду, что `useRef` не уведомляет вас, когда изменяется его содержимое. Вы можете изменять и обновлять `current`. Мутирование свойства `.current` не вызывает повторный рендер.

Если вы хотите запустить некоторый код, когда React присоединяет или отсоединяет реф к узлу DOM, вы можете использовать колбэк-реф.

По умолчанию React не позволяет компоненту получить доступ к узлам DOM других компонентов. Даже для собственных дочерних! Для этого есть `forwardRef`.

~~~
function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert("You clicked " + ref.current + " times!");
  }

  return <button onClick={handleClick}>Click me!</button>;
}
~~~

### Пример: создание секундомера

Вы можете комбинировать `refs` и состояние в одном компоненте. Например, давайте сделаем секундомер, который пользователь может запускать или останавливать нажатием кнопки.

~~~
function Stopwatch() {
  const [startTime, setStartTime] = useState<number | null>(null);
  const [now, setNow] = useState<number | null>(null);
  const intervalRef = useRef<ReturnType<typeof setInterval> | undefined>(
    undefined
  );

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
~~~

Поскольку идентификатор интервала не используется для рендеринга, вы можете сохранить его в `ref`.

### Различия между `refs` и состоянием

| `refs` | состояние |
|---|---|
| `useRef(initialValue)` возвращает `{ current: initialValue }`	| `useState(initialValue)` возвращает текущее значение переменной состояния и функцию установки состояния (`[value, setValue]`) |
| Не вызывает повторный рендеринг при его изменении.	| Запускает повторный рендеринг при его изменении. |
| Изменяемый — вы можете изменять и обновлять значение `current` вне процесса рендеринга.	| «Неизменный» — вы должны использовать функцию настройки состояния, чтобы изменить переменные состояния, чтобы поставить в очередь повторный рендеринг. |
| Вы не должны читать (или записывать) значение `current` во время рендеринга.	| Вы можете прочитать состояние в любое время. Однако каждый рендер имеет свой собственный снимок состояния, который не меняется. |

### Когда использовать `refs`

Вот несколько таких редких ситуаций:

* Хранение идентификаторов тайм-аута.
* Хранение элементов DOM и управление ими.
* Хранение других объектов, которые не нужны для расчета JSX.

Если вашему компоненту нужно хранить какое-то значение, но это не влияет на логику рендеринга, выберите `refs`.

### Рекомендации для `refs`

Следование этим принципам сделает ваши компоненты более предсказуемыми:

* Относитесь к `refs` как к аварийному люку. `Refs` полезны при работе с внешними системами или API-интерфейсами браузера. Если большая часть логики вашего приложения и потока данных зависит от `refs`, вы можете пересмотреть свой подход.
* Не читайте и не записывайте `ref.current` во время рендеринга. Если во время рендеринга требуется некоторая информация, используйте вместо этого состояние. Поскольку React не знает, когда изменяется `ref.current`, даже чтение его во время рендеринга затрудняет прогнозирование поведения вашего компонента. (Единственным исключением является такой код, как `if (!ref.current) ref.current = new Thing()`, который устанавливает `ref` только один раз во время первого рендеринга.)

Ограничения состояния React не распространяются на `refs`. Например, состояние действует как снимок для каждого рендеринга и не обновляется синхронно. Но когда вы изменяете текущее значение `ref`, оно немедленно меняется:

~~~
ref.current = 5;
console.log(ref.current); // 5
~~~

Это связано с тем, что сам `ref` является обычным объектом JavaScript и ведет себя как таковой.

Вам также не нужно беспокоиться о том, чтобы избежать мутации при работе с `ref`. Пока объект, который вы мутируете, не используется для рендеринга, React не волнует, что вы делаете с `ref` или ее содержимым.

### Управление DOM с помощью `refs`

Иногда вам может понадобиться доступ к элементам DOM, управляемым React, например, чтобы сфокусировать узел, прокрутить его или измерить его размер и положение. В React нет встроенного способа сделать это, поэтому вам понадобится `ref` на узел DOM.

#### Пример. Фокусировка ввода текста

~~~
function App() {
  const inputEl = useRef<HTMLInputElement>(null);
  const onButtonClick = () => {
    (inputEl.current as HTMLInputElement).focus();
  };

  return (
    <>
      <input title="inputEl" ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
~~~

#### Пример: прокрутка до элемента

~~~
function CatFriends() {
  const firstCatRef = useRef<HTMLImageElement>(null);
  const secondCatRef = useRef<HTMLImageElement>(null);
  const thirdCatRef = useRef<HTMLImageElement>(null);

  function handleScrollToFirstCat() {
    firstCatRef.current!.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current!.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current!.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>Tom</button>
        <button onClick={handleScrollToSecondCat}>Maru</button>
        <button onClick={handleScrollToThirdCat}>Jellylorum</button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
~~~

### `forwardRef`

Если вы попытаетесь поместить `ref` на свой собственный компонент, например `<MyInput />`, по умолчанию вы получите `null`.

Это происходит потому, что по умолчанию React не позволяет компоненту получить доступ к узлам DOM других компонентов.

Вместо этого компоненты, которые хотят выставить свои узлы DOM, должны согласиться на такое поведение.

~~~
const MyInput = forwardRef<HTMLInputElement>((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  function handleClick() {
    inputRef.current!.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
~~~

### Когда React прикрепляет `refs`

В React каждое обновление разделено на две фазы:

* Во время фазы `render` React вызывает ваши компоненты, чтобы выяснить, что должно быть на экране.
* Во время фазы `commit` React применяет изменения к DOM.

Как правило, вы не хотите получать доступ к `refs` во время рендеринга. Это касается и `refs`, содержащих узлы DOM. Во время первого рендера узлы DOM еще не созданы, поэтому `ref.current` будут `null`.

React устанавливает `ref.current` во время фазы `commit`. Перед обновлением DOM React устанавливает затронутые значения `ref.current` в `null`. После обновления DOM React немедленно устанавливает их в соответствующие узлы DOM.

### Лучшие практики для манипулирования DOM с помощью `refs`

Если вы придерживаетесь неразрушающих действий, таких как фокусировка и прокрутка, у вас не должно возникнуть никаких проблем. Однако, если вы попытаетесь изменить DOM вручную, вы рискуете столкнуться с изменениями, которые вносит React.

Чтобы проиллюстрировать эту проблему рассмотрим пример:

~~~
function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef<HTMLParagraphElement>(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}
      >
        Toggle with setState
      </button>
      <button
        onClick={() => {
          ref.current!.remove();
        }}
      >
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
~~~

Попробуйте нажать `«Toggle with setState»` несколько раз. Сообщение должно исчезнуть и появиться снова. Затем нажмите `«Remove from the DOM»`. Это принудительно удалит его. Наконец, нажмите `«Toggle with setState»`.

После того, как вы вручную удалили элемент DOM, попытка снова использовать `setState` для отображения приведет к сбою. Это потому, что вы изменили DOM, а React не знает, как продолжать правильно управлять им.

Избегайте изменения узлов DOM, управляемых React.

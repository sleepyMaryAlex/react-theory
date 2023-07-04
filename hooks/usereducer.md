# ?useReducer

Компоненты с большим количеством обновлений состояния, разбросанных по множеству обработчиков событий, могут оказаться перегруженными. В этих случаях вы можете объединить всю логику обновления состояния за пределами вашего компонента в одной функции, называемой `reducer`.

`useReducer` — это React Hook, который позволяет вам добавить `reducer` к вашему компоненту.

~~~
const [state, dispatch] = useReducer(reducer, initialArg, init?)
~~~

* `reducer`: функция редьюсера, которая указывает, как обновляется состояние. Она должна быть чистой, должна принимать состояние и действие в качестве аргументов и должна возвращать следующее состояние. Состояние и действие могут быть любого типа.
* `initialArg`: значение, из которого вычисляется начальное состояние. Это может быть значение любого типа. Как из него вычисляется начальное состояние, зависит от следующего аргумента `init`.
* необязательный `init` : функция инициализатора, которая должна возвращать начальное состояние. Если он не указан, устанавливается начальное состояние `initialArg`. В противном случае начальное состояние устанавливается на результат вызова `init(initialArg)`.

`useReducer` возвращает массив ровно с двумя значениями:

1. Текущее состояние. Во время первого рендеринга устанавливается `init(initialArg)` или `initialArg` (если нет `init`).
2. Функция `dispatch`, которая позволяет обновить состояние до другого значения и запустить повторный рендеринг. Функции `dispatch` не имеют возвращаемого значения.

App.tsx
~~~
export const StateDispatch = createContext<React.Dispatch<{
  type: string;
}> | null>(null);

function reducer(state: { count: number }, action: { type: string }) {
  switch (action.type) {
    case "increment": {
      return { count: state.count + 1 };
    }
    case "decrement": {
      return { count: state.count - 1 };
    }
    default: {
      return { count: state.count };
    }
  }
}

function App() {
  const initialState = { count: 0 };
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateDispatch.Provider value={dispatch}>
      <Counter count={state.count} />
    </StateDispatch.Provider>
  );
}

export default App;
~~~

Counter.tsx
~~~
function Counter(props: { count: number }) {
  const dispatch = useContext(StateDispatch) as React.Dispatch<{
    type: string;
  }>;
  return (
    <>
      Count: {props.count}
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}

export default Counter;
~~~

Для больших деревьев компонентов мы рекомендуем передавать вниз функцию `dispatch` из хука `useReducer` через контекст, как показано выше.

`action` - это обычный объект JavaScript. Вы сами решаете, что в него поместить, но в целом он должен содержать минимум информации о том, что произошло. По соглашению принято давать ему строку `type`, описывающую, что произошло, и передавать любую дополнительную информацию в другие поля.

Функция `reducer` - это место, где вы поместите свою логику состояния. Он принимает два аргумента, текущее состояние и объект `action`, и возвращает следующее состояние. Вы можете объявить ее вне вашего компонента. Принято использовать оператор `switch` внутри `reducer`. Функции `reducer` должны обновлять объекты и массивы без мутаций.

Вы можете передать функцию `init` в качестве третьего аргумента. Начальное состояние будет установлено равным результату вызова `init(initialArg)`. Это позволяет извлечь логику для расчёта начального состояния за пределы `reducer`. Это также удобно для сброса состояния позже в ответ на действие:

App.tsx
~~~
export const StateDispatch = createContext<React.Dispatch<{
  type: string;
}> | null>(null);

function init(initialCount: number) {
  return { count: initialCount };
}

function reducer(
  state: { count: number },
  action: { type: string; payload?: number }
) {
  switch (action.type) {
    case "increment": {
      return { count: state.count + 1 };
    }
    case "decrement": {
      return { count: state.count - 1 };
    }
    case "reset": {
      return init(action.payload as number);
    }
    default: {
      return { count: state.count };
    }
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, 5, init); // вместо 5 может быть переменная из пропсов, например
  return (
    <StateDispatch.Provider value={dispatch}>
      <Counter count={state.count} />
    </StateDispatch.Provider>
  );
}

export default App;
~~~

Counter.tsx
~~~
function Counter(props: { count: number }) {
  const dispatch = useContext(StateDispatch) as React.Dispatch<{
    type: string;
    payload?: number;
  }>;
  return (
    <>
      Count: {props.count}
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "reset", payload: 0 })}>
        Reset
      </button>
    </>
  );
}

export default Counter;
~~~

### Написание кратких редьюсеров с помощью `Immer`

Как и при обновлении объектов и массивов в обычном состоянии, вы можете использовать библиотеку Immer, чтобы сделать редукторы более краткими. Здесь `useImmerReducer` позволяет изменить состояние с помощью `push` или `arr[i] =` присваивания:

~~~
function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    // ...
  }
}
~~~

~~~
const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);
~~~

Рекомендуется заключать каждый caseблок в фигурные скобки `{}`, чтобы переменные, объявленные внутри разных `case`, не конфликтовали друг с другом.

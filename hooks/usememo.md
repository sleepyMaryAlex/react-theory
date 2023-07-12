# ?useMemo

`useMemo` - это React Hook, который позволяет кэшировать результат вычислений между повторными рендерингами.

`useMemo` и `useCallback` похожи. Основное отличие состоит в том, что `useMemo` возвращает мемоизированное значение, а `useCallback` возвращает мемоизированную функцию.

Ниже функция `expensiveCalculation` запускается при каждом рендере:

App.tsx
~~~
const App = () => {
  const [count, setCount] = useState(0);
  const [todos, setTodos] = useState<string[]>([]);
  const calculation = expensiveCalculation(count);

  const increment = () => {
    setCount((c) => c + 1);
  };

  const addTodo = () => {
    setTodos((t) => [...t, "New Todo"]);
  };

  return (
    <div>
      <div>
        <h2>My Todos</h2>
        {todos.map((todo, index) => {
          return <p key={index}>{todo}</p>;
        })}
        <button onClick={addTodo}>Add Todo</button>
      </div>
      <hr />
      <div>
        Count: {count}
        <button onClick={increment}>+</button>
        <h2>Expensive Calculation</h2>
        {calculation}
      </div>
    </div>
  );
};
~~~

expensiveCalculation.ts
~~~
const expensiveCalculation = (num:  number) => {
  console.log("Calculating...");
  for (let i = 0; i < 1000000000; i++) {
    num += 1;
  }
  return num;
};
~~~

Чтобы решить эту проблему с производительностью, мы можем использовать хук `useMemo` для запоминания функции `expensiveCalculation`. Хук `useMemo` принимает второй параметр для объявления зависимостей. Теперь дорогостоящая функция будет работать только тогда, когда ее зависимости изменились.

~~~
const calculation = useMemo(() => expensiveCalculation(count), [count]);
~~~

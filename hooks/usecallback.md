# ?useCallback

`useCallback—` это React Hook, который позволяет кэшировать определение функции между повторными рендерингами.

`useCallback(fn, dependencies)`

* `fn`: значение функции, которое вы хотите кэшировать.
* `dependencies`: список всех значений, на которые ссылается код `fn`.

При начальном рендеринге `useCallback` возвращает переданную вами функцию `fn`.

При последующих рендерах он либо вернет уже сохраненную функцию `fn` из последнего рендера (если зависимости не изменились), либо вернет функцию `fn`, которую вы передали во время этого рендера.

### Пропуск повторного рендеринга компонентов

По умолчанию, когда компонент перерисовывается, React рекурсивно перерисовывает все его дочерние элементы.

App.tsx
~~~
function App() {
  const handleSubmit = () => {
    console.log("Submit");
  };

  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <button onClick={handleClick}>Increase</button>
      <p>Count: {count}</p>
      <Form onSubmit={handleSubmit} />
    </div>
  );
}

export default App;
~~~

Form.tsx
~~~
const Form = memo(({ onSubmit }: { onSubmit: () => void }) => {
  console.log("Render");
  return <button onClick={onSubmit}>Submit</button>;
});

export default Form;
~~~

В JavaScript `function() {}` или `() => {}` всегда создает другую функцию, подобно тому, как литерал объекта `{}` всегда создает новый объект. Обычно это не было бы проблемой, но это означает, что пропсы `Form` никогда не будут прежними, и оптимизация `memo` не будет работать. Здесь пригодится `useCallback`:

~~~
const handleSubmit = useCallback(() => {
  console.log("Submit");
}, []);
~~~

Оборачивая `handleSubmit` в `useCallback`, вы гарантируете, что это одна и та же функция между повторными рендерингами (до тех пор, пока не изменятся зависимости).

### Оптимизация пользовательского хука

Если вы пишете собственный хук, рекомендуется обернуть все функции, которые он возвращает, в `useCallback`:

~~~
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
~~~

Это гарантирует, что потребители вашего хука смогут при необходимости оптимизировать свой собственный код.

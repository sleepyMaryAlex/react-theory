# ?Small react app: form, button, results list

~~~
import { useState } from "react";

let nextId = 0;

export default function List() {
  const [value, setValue] = useState<string>("");
  const [todos, setTodos] = useState<{ id: number; value: string }[]>([]);

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setTodos([...todos, { id: nextId++, value: value }]);
    setValue("");
  }

  return (
    <>
      <h1>My Todos:</h1>
      <form onSubmit={handleSubmit}>
        <input value={value} onChange={(e) => setValue(e.target.value)} />
        <button type="submit">
          Add
        </button>
      </form>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.value}</li>
        ))}
      </ul>
    </>
  );
}
~~~

### event.preventDefault

Этот метод предотвращает действия по умолчанию, которые браузеры выполняют при запуске события.

Когда пользователь отправляет форму (нажата кнопка отправки), действие формы по умолчанию — отправить данные формы на URL-адрес, который обрабатывает данные.

Элементы формы имеют атрибуты `action` и `method`, указывающие URL-адрес для отправки формы и тип запроса соответственно.

Если эти атрибуты не указаны, URL-адрес по умолчанию — это текущий URL-адрес, по которому была отправлена ​​форма, и метод — `get`.

~~~
export default function List() {
  const [value, setValue] = useState<string>("");

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const target = e.target as HTMLFormElement;
    console.log(target.method); // get
    console.log(target.action); // http://localhost:3000/?
    console.log(target.todo.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={value}
        name="todo"
        onChange={(e) => setValue(e.target.value)}
      />
      <button type="submit">Add</button>
    </form>
  );
}
~~~

Использование метода `preventDefault()` в обработчике события отправки формы позволяет предотвратить стандартное поведение отправки формы на сервер и перезагрузку страницы.

Когда вызывается `event.preventDefault()` в обработчике события отправки формы, это останавливает выполнение стандартных действий, связанных с отправкой формы браузером. Таким образом, данные формы не будут автоматически отправлены на сервер, и страница не будет перезагружена.

После вызова `preventDefault()`, вы можете использовать JavaScript для обработки данных формы и выполнения других пользовательских действий. Например, вы можете собрать данные формы, отправить их на сервер асинхронным запросом (например, с использованием AJAX), обновить содержимое страницы динамически или выполнить другую пользовательскую логику.

Обратите внимание, что событие отправки запускается для самого элемента `<form>`, а не для какой-либо `<button>` или `<input type="submit">` внутри него.

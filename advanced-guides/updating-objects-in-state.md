# ?Updating objects in state

Состояние может содержать любое значение JavaScript, включая объекты. Но вы не должны напрямую изменять объекты, которые вы держите в состоянии React. Вместо этого, когда вы хотите обновить объект, вам нужно создать новый (или сделать копию существующего), а затем установить состояние для использования этой копии.

~~~
// Вставка ключа
// Плохо
this.state.obj.key = 'bar';
// Хорошо
this.setState({
  obj: {
    ...this.state.obj,
    key: 'bar'
  }
})

// Удаление ключа
// Плохое
удаление this.state.obj.key;
// Хорошее
const {key, ...newObj} = this.state.objthis.setState({
  obj: newObj
})
~~~

Обратите внимание, что синтаксис расширения `...` «поверхностный» — он копирует элементы только на один уровень вглубь. Это делает его быстрым, но это также означает, что если вы хотите обновить вложенное свойство, вам придется использовать его более одного раза.

~~~
function Form() {
  const [person, setPerson] = useState({
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
    },
  });

  function handleNameChange(e: React.ChangeEvent<HTMLInputElement>) {
    setPerson({
      ...person,
      name: e.target.value,
    });
  }

  function handleTitleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value,
      },
    });
  }

  function handleCityChange(e: React.ChangeEvent<HTMLInputElement>) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value,
      },
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {" by "}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
    </>
  );
}
~~~

Также можно использовать библиотеку `Immer`.

Чтобы попробовать `Immer`:

1. Запустите `npm install use-immer`, чтобы добавить Immer в качестве зависимости
2. Затем замените `import { useState } from 'react'` на `import { useImmer } from 'use-immer'`

~~~
import { useImmer } from "use-immer";

function Form() {
  const [person, updatePerson] = useImmer({
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
    },
  });

  function handleNameChange(e: React.ChangeEvent<HTMLInputElement>) {
    updatePerson((draft) => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e: React.ChangeEvent<HTMLInputElement>) {
    updatePerson((draft) => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e: React.ChangeEvent<HTMLInputElement>) {
    updatePerson((draft) => {
      draft.artwork.city = e.target.value;
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {" by "}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
    </>
  );
}
~~~

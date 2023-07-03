# ?Updating arrays in state

Массивы изменяемы в JavaScript, но вы должны обращаться с ними как с неизменяемыми, когда сохраняете их в состоянии. Как и в случае с объектами, когда вы хотите обновить массив, хранящийся в состоянии, вам нужно создать новый (или сделать копию существующего), а затем установить состояние для использования нового массива.

| | избежать (мутирует массив) | предпочесть (возвращает новый массив) |
|---|---|---|
| добавление | `push`, `unshift`	| `concat`, синтаксис распространения `[...arr]` |
| удаление | `pop`, `shift`, `splice`	| `filter`, `slice` |
| замена | `splice`, `arr[i] = ...присвоение`	| `map` |
| сортировка | `reverse`, `sort` | сначала скопируйте массив |

В качестве альтернативы можно использовать библиотеку `Immer`, который позволяет использовать методы из обоих столбцов.

`Immer` — это популярная библиотека, которая позволяет вам писать с использованием удобного, но изменяющегося синтаксиса и заботится о создании копий для вас. С `Immer` код, который вы пишете, выглядит так, будто вы «нарушаете правила» и мутируете объект, но в отличие от обычной мутации, она не перезаписывает прошлое состояние!

К сожалению, `slice` и `splice` называются почти одинаково, но сильно отличаются:

* `slice` позволяет копировать массив или его часть.
* `splice` изменяет массив (чтобы вставлять или удалять элементы).

~~~
// Добавляем в конец массива
// Плохо
this.state.arr.push('foo');
// Хорошо
this.setState({
  arr: [...this.state.arr, 'foo']
})

// Замена item в середине
// Плохо
this.state.arr[3] = 'foo';
//Хорошо
this.setState({
  arr: this.state.arr.map((item, index) => index === 3 ? 'foo' : item)
})

//Добавление в середину массива
// Плохо
this.state.arr.splice(3, 0, 'foo');
// Хорошо
this.setState({
  arr: [...this.state.arr.slice(0, 1), "foo", ...this.state.arr.slice(1)],
});

// Удаление из массива
// Плохо
this.state.arr.splice(2, 1)
// Хорошо
this.setState({
  arr: this.state.arr.filter((item, index) => index !== 2 )
})
~~~

JavaScript методы `reverse()` и `sort()` мутируют исходный массив, поэтому вы не можете использовать их напрямую. Однако вы можете сначала скопировать массив, а затем внести в него изменения.

~~~
function handleClick() {
  const nextList = [...list];
  nextList.reverse();
  setList(nextList);
}
~~~

Однако, даже если вы скопируете массив, вы не сможете напрямую изменить существующие элементы внутри него. Это связано с тем, что копирование неглубокое — новый массив будет содержать те же элементы, что и исходный. Поэтому, если вы изменяете объект внутри скопированного массива, вы изменяете существующее состояние. Например, такой код является проблемой.

~~~
const nextList = [...list];
nextList[0].seen = true; // Problem: mutates list[0]
setList(nextList);
~~~

Хотя `nextList` и `list` являются двумя разными массивами `nextList[0]` и `list[0]` указывают на один и тот же объект. Таким образом, изменяясь `nextList[0].seen`, вы также меняетесь `list[0].seen`. Это мутация состояния, которой следует избегать!

### Обновление объектов внутри массивов

В этом примере два отдельных списка иллюстраций имеют одинаковое начальное состояние. Предполагается, что они изолированы, но из-за мутации их состояние случайно становится общим, и установка флажка в одном списке влияет на другой список:

BucketList.tsx
~~~
export interface IArtwork {
  id: number;
  title: string;
  seen: boolean;
}

const initialList = [
  { id: 0, title: "Big Bellies", seen: false },
  { id: 1, title: "Lunar Landscape", seen: false },
  { id: 2, title: "Terracotta Army", seen: true },
];

function BucketList() {
  const [myList, setMyList] = useState<IArtwork[]>(initialList);
  const [yourList, setYourList] = useState<IArtwork[]>(initialList);

  function handleToggleMyList(artworkId: number, nextSeen: boolean) {
    const myNextList = [...myList];
    const artwork = myNextList.find((a) => a.id === artworkId);
    (artwork as IArtwork).seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId: number, nextSeen: boolean) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find((a) => a.id === artworkId);
    (artwork as IArtwork).seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}
~~~

ItemList.tsx
~~~
function ItemList({
  artworks,
  onToggle,
}: {
  artworks: IArtwork[];
  onToggle: (artworkId: number, nextSeen: boolean) => void;
}) {
  return (
    <ul>
      {artworks.map((artwork) => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={(e) => {
                onToggle(artwork.id, e.target.checked);
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
~~~

Проблема в таком коде:

~~~
const myNextList = [...myList];
const artwork = myNextList.find((a) => a.id === artworkId);
(artwork as IArtwork).seen = nextSeen;
setMyList(myNextList);
~~~

Хотя сам массив `myNextList` новый, сами элементы такие же, как и в исходном массиве `myList`. Таким образом, изменение `artwork.seen` изменяет исходный элемент. Этот элемент также находится в `yourList`, что вызывает ошибку. О таких ошибках может быть трудно думать, но, к счастью, они исчезают, если вы избегаете мутирующего состояния.

Вы можете использовать `map` для замены старого элемента его обновленной версией без изменения.

~~~
function handleToggleMyList(artworkId: number, nextSeen: boolean) {
  setMyList(
    myList.map((artwork) => {
      if (artwork.id === artworkId) {
        return { ...artwork, seen: nextSeen };
      } else {
        return artwork;
      }
    })
  );
}

function handleToggleYourList(artworkId: number, nextSeen: boolean) {
  setYourList(
    yourList.map((artwork) => {
      if (artwork.id === artworkId) {
        return { ...artwork, seen: nextSeen };
      } else {
        return artwork;
      }
    })
  );
}
~~~

При таком подходе ни один из существующих элементов состояния не мутируется, а ошибка исправлена.

Как правило, вам следует изменять только те объекты, которые вы только что создали.

### Краткая логика обновления с помощью `Immer`

Вот пример `Art Bucket List`, переписанный с помощью `Immer`:

~~~
function BucketList() {
  const [myList, updateMyList] = useImmer(initialList);
  const [yourList, updateYourList] = useImmer(initialList);

  function handleToggleMyList(artworkId: number, nextSeen: boolean) {
    updateMyList((draft) => {
      const artwork = draft.find((a) => a.id === artworkId);
      (artwork as IArtwork).seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId: number, nextSeen: boolean) {
    updateYourList((draft) => {
      const artwork = draft.find((a) => a.id === artworkId);
      (artwork as IArtwork).seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}
~~~

Обратите внимание, как с `Immer` мутация вроде `artwork.seen = nextSeen` теперь в порядке.

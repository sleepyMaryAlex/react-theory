# ?Choosing the state structure

Когда вы пишете компонент, который содержит какое-то состояние, придется выбирать, сколько переменных состояния использовать и какой должна быть форма их данных.

Есть несколько принципов:

1. __Состояние, связанное с группой.__ Если вы всегда одновременно обновляете две или более переменных состояния, рассмотрите возможность их объединения в одну переменную состояния.
2. __Избегайте противоречий в состоянии.__ Когда состояние структурировано таким образом, что несколько частей состояния могут противоречить и «не согласовываться» друг с другом, вы оставляете место для ошибок. Постарайтесь избежать этого.

~~~
function FeedbackForm() {
  const [text, setText] = useState("");
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <br />
      <button disabled={isSending} type="submit">
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  });
}
~~~

Хотя этот код работает, он оставляет дверь открытой для «невозможных» состояний. Например, если вы забудете вызвать `setIsSent` и `setIsSending` вместе, вы можете оказаться в ситуации, когда оба `isSending` и `isSent` могут быть одновременно `true`.

Поскольку `isSending` и `isSent` никогда не должны быть одновременно `true`, лучше заменить их одной переменной состояния `status`, которая может принимать одно из трех допустимых состояний: `'typing'` (начальное), `'sending'` и `'sent'`:

~~~
const [text, setText] = useState("");
const [status, setStatus] = useState("typing");
~~~

3. __Избегайте избыточного состояния.__ Если вы можете вычислить некоторую информацию из свойств компонента или его существующих переменных состояния во время рендеринга, вы не должны помещать эту информацию в состояние этого компонента.

Например, форма имеет три переменные состояния: `firstName`, `lastName` и `fullName`. Однако `fullName` избыточен. Вы всегда можете рассчитать `fullName` из `firstName` и `lastName` во время рендеринга, поэтому удалите его из состояния.

4. __Избегайте дублирования состояния.__ Когда одни и те же данные дублируются между несколькими переменными состояния или внутри вложенных объектов, их сложно синхронизировать. Уменьшите дублирование, когда сможете.

~~~
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(
  items[0]
);
~~~

Компонент сохраняет выбранный элемент как объект в переменной состояния `selectedItem`. Однако это не очень хорошо: содержимое `selectedItem` — это тот же объект, что и один из элементов внутри списка `items`. Это означает, что информация о самом предмете дублируется в двух местах.

Вместо объекта `selectedItem` (который создает дублирование объектов внутри элементов) лучше сохранить `selectedId` в состоянии, а затем получаете `selectedItem` путем поиска в массиве элементов элемента с этим `id`:

~~~
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(0);

const selectedItem = items.find((item) => item.id === selectedId);
~~~

5. __Избегайте глубоко вложенных состояний.__ Глубоко иерархическое состояние не очень удобно обновлять. По возможности предпочтительнее структурировать состояние в плоском виде.

Представьте план путешествия, состоящий из планет, континентов и стран. У вас может возникнуть соблазн структурировать его состояние с помощью вложенных объектов и массивов, как в этом примере:

places.js
~~~
export const initialTravelPlan = {
  id: 0,
  title: "(Root)",
  childPlaces: [
    {
      id: 1,
      title: "Earth",
      childPlaces: [
        {
          id: 2,
          title: "Africa",
          childPlaces: [
            {
              id: 3,
              title: "Botswana",
              childPlaces: [],
            },
            {
              id: 4,
              title: "Egypt",
              childPlaces: [],
            },
            {
              id: 5,
              title: "Kenya",
              childPlaces: [],
            },
            {
              id: 6,
              title: "Madagascar",
              childPlaces: [],
            },
            {
              id: 7,
              title: "Morocco",
              childPlaces: [],
            },
            {
              id: 8,
              title: "Nigeria",
              childPlaces: [],
            },
            {
              id: 9,
              title: "South Africa",
              childPlaces: [],
            },
          ],
        },
        {
          id: 10,
          title: "Americas",
          childPlaces: [
            {
              id: 11,
              title: "Argentina",
              childPlaces: [],
            },
            {
              id: 12,
              title: "Brazil",
              childPlaces: [],
            },
            {
              id: 13,
              title: "Barbados",
              childPlaces: [],
            },
            {
              id: 14,
              title: "Canada",
              childPlaces: [],
            },
            {
              id: 15,
              title: "Jamaica",
              childPlaces: [],
            },
            {
              id: 16,
              title: "Mexico",
              childPlaces: [],
            },
            {
              id: 17,
              title: "Trinidad and Tobago",
              childPlaces: [],
            },
            {
              id: 18,
              title: "Venezuela",
              childPlaces: [],
            },
          ],
        },
        {
          id: 19,
          title: "Asia",
          childPlaces: [
            {
              id: 20,
              title: "China",
              childPlaces: [],
            },
            {
              id: 21,
              title: "Hong Kong",
              childPlaces: [],
            },
            {
              id: 22,
              title: "India",
              childPlaces: [],
            },
            {
              id: 23,
              title: "Singapore",
              childPlaces: [],
            },
            {
              id: 24,
              title: "South Korea",
              childPlaces: [],
            },
            {
              id: 25,
              title: "Thailand",
              childPlaces: [],
            },
            {
              id: 26,
              title: "Vietnam",
              childPlaces: [],
            },
          ],
        },
        {
          id: 27,
          title: "Europe",
          childPlaces: [
            {
              id: 28,
              title: "Croatia",
              childPlaces: [],
            },
            {
              id: 29,
              title: "France",
              childPlaces: [],
            },
            {
              id: 30,
              title: "Germany",
              childPlaces: [],
            },
            {
              id: 31,
              title: "Italy",
              childPlaces: [],
            },
            {
              id: 32,
              title: "Portugal",
              childPlaces: [],
            },
            {
              id: 33,
              title: "Spain",
              childPlaces: [],
            },
            {
              id: 34,
              title: "Turkey",
              childPlaces: [],
            },
          ],
        },
        {
          id: 35,
          title: "Oceania",
          childPlaces: [
            {
              id: 36,
              title: "Australia",
              childPlaces: [],
            },
            {
              id: 37,
              title: "Bora Bora (French Polynesia)",
              childPlaces: [],
            },
            {
              id: 38,
              title: "Easter Island (Chile)",
              childPlaces: [],
            },
            {
              id: 39,
              title: "Fiji",
              childPlaces: [],
            },
            {
              id: 40,
              title: "Hawaii (the USA)",
              childPlaces: [],
            },
            {
              id: 41,
              title: "New Zealand",
              childPlaces: [],
            },
            {
              id: 42,
              title: "Vanuatu",
              childPlaces: [],
            },
          ],
        },
      ],
    },
    {
      id: 43,
      title: "Moon",
      childPlaces: [
        {
          id: 44,
          title: "Rheita",
          childPlaces: [],
        },
        {
          id: 45,
          title: "Piccolomini",
          childPlaces: [],
        },
        {
          id: 46,
          title: "Tycho",
          childPlaces: [],
        },
      ],
    },
    {
      id: 47,
      title: "Mars",
      childPlaces: [
        {
          id: 48,
          title: "Corn Town",
          childPlaces: [],
        },
        {
          id: 49,
          title: "Green Hill",
          childPlaces: [],
        },
      ],
    },
  ],
};
~~~

App.js
~~~
const [plan, setPlan] = useState(initialTravelPlan);
~~~

Теперь предположим, что вы хотите добавить кнопку для удаления места, которое вы уже посетили. Как бы вы это сделали? Обновление вложенного состояния включает в себя создание копий объектов от той части, которая была изменена. Удаление глубоко вложенного места потребует копирования всей его родительской цепочки мест.

Если состояние слишком вложенное, чтобы его можно было легко обновить, подумайте о том, чтобы сделать его «плоским».

places.js
~~~
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 43, 47],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 27, 35]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  // ...
};
~~~

Целью этих принципов является упрощение обновления состояния без внесения ошибок.

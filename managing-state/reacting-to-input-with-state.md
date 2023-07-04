# ?Reacting to input with state

React предоставляет декларативный способ управления пользовательским интерфейсом. Вместо прямого управления отдельными частями пользовательского интерфейса вы описываете различные состояния, в которых может находиться ваш компонент, и переключаетесь между ними в ответ на действия пользователя.

### Сравнение декларативного пользовательского интерфейса с императивным

В императивном программировании вы должны написать точные инструкции для управления пользовательским интерфейсом в зависимости от того, что только что произошло.

В этом примере императивного программирования пользовательского интерфейса форма создается без использования React. Он использует только DOM браузера:

index.js
~~~
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = "none";
}

function show(el) {
  el.style.display = "";
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() == "istanbul") {
        resolve();
      } else {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      }
    }, 1500);
  });
}

let form = document.getElementById("form");
let textarea = document.getElementById("textarea");
let button = document.getElementById("button");
let loadingMessage = document.getElementById("loading");
let errorMessage = document.getElementById("error");
let successMessage = document.getElementById("success");
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
~~~

index.html
~~~
<form id="form">
  <h2>City quiz</h2>
  <p>
    What city is located on two continents?
  </p>
  <textarea id="textarea"></textarea>
  <br />
  <button id="button" disabled>Submit</button>
  <p id="loading" style="display: none">Loading...</p>
  <p id="error" style="display: none; color: red;"></p>
</form>
<h1 id="success" style="display: none">That's right!</h1>

<style>
  * {
    box-sizing: border-box;
  }

  body {
    font-family: sans-serif;
    margin: 20px;
    padding: 0;
  }
</style>
~~~

Императивное управление пользовательским интерфейсом работает достаточно хорошо для отдельных примеров, но в более сложных системах управлять им становится экспоненциально сложнее.

Представьте, что вы обновляете страницу, полную различных форм, подобных этой. Добавление нового элемента пользовательского интерфейса или нового взаимодействия потребует тщательной проверки всего существующего кода, чтобы убедиться, что вы не внесли ошибку (например, забыли что-то показать или скрыть).

React был создан, чтобы решить эту проблему.

В React вы не управляете пользовательским интерфейсом напрямую — это означает, что вы не включаете, не отключаете, не показываете и не скрываете компоненты напрямую. Вместо этого вы объявляете, что хотите показать, а React выясняет, как обновить пользовательский интерфейс.

Подумайте о том, чтобы сесть в такси и сказать водителю, куда вы хотите ехать, вместо того, чтобы указывать ему, где именно повернуть. Доставить вас туда — работа водителя, и они могут даже знать некоторые короткие пути, о которых вы не подумали!

### Декларативное мышление

Мы видели, как императивно реализовать форму выше. Чтобы лучше понять, как мыслить в React, выполните повторную реализацию этого пользовательского интерфейса в React:

1. Определите различные визуальные состояния вашего компонента.
2. Определите , что вызывает эти изменения состояния.
3. Представьте состояние в памяти, используя `useState`.
4. Удалите все несущественные переменные состояния.
5. Подключите обработчики событий, чтобы установить состояние.

~~~
function Form() {
  const [answer, setAnswer] = useState("");
  const [error, setError] = useState<Error | unknown | null>(null);
  const [status, setStatus] = useState("typing");

  if (status === "success") {
    return <h1>That's right!</h1>;
  }

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setStatus("submitting");
    try {
      await submitForm(answer);
      setStatus("success");
    } catch (err) {
      setStatus("typing");
      setError(err);
    }
  }

  function handleTextareaChange(e: React.ChangeEvent<HTMLTextAreaElement>) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === "submitting"}
        />
        <br />
        <button disabled={answer.length === 0 || status === "submitting"}>
          Submit
        </button>
        {error !== null && <p className="Error">{(error as Error).message}</p>}
      </form>
    </>
  );
}

function submitForm(answer: string) {
  // Pretend it's hitting the network.
  return new Promise<void>((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== "lima";
      if (shouldError) {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      } else {
        resolve();
      }
    }, 1500);
  });
}
~~~

Хотя этот код длиннее исходного императивного примера, он намного менее хрупок. Выражение всех взаимодействий в виде изменений состояния позволяет позже вводить новые визуальные состояния, не нарушая существующие.

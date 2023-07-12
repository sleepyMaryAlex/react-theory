# ?useImperativeHandle

`useImperativeHandle(ref, createHandle, dependencies?)`

Параметры:

* `ref`: ссылка, которую вы получили в качестве второго аргумента от функции рендеринга `forwardRef`.
* `createHandle`: функция, которая не принимает аргументов и возвращает дескриптор `ref`, которую вы хотите предоставить. Этот дескриптор `ref` может иметь любой тип. Обычно вы возвращаете объект с методами, которые хотите предоставить.
* `optional dependencies`: список всех реактивных значений, на которые ссылается код `createHandle`.

`useImperativeHandle` возвращает `undefined`.

### Применение

Например, предположим, что вы не хотите показывать весь DOM-узел `<input>`, но хотите открыть два его метода: `focus` и `scrollIntoView`. Для этого держите настоящий DOM браузера в отдельном `ref`. Затем используйте `useImperativeHandle`, чтобы предоставить дескриптор только тех методов, которые вы хотите, чтобы родительский компонент вызывал:

App.tsx
~~~
import { useRef } from "react";
import MyInput from "./MyInput";

export default function Form() {
  const ref = useRef<HTMLInputElement>(null);

  function handleClick() {
    ref.current?.focus();
    // This won't work because the DOM node isn't exposed:
    // ref.current!.style.opacity = "0.5";
  }

  return (
    <form>
      <MyInput ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
~~~

myInput.tsx
~~~
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current?.focus();
        },
        scrollIntoView() {
          inputRef.current?.scrollIntoView();
        },
      };
    },
    []
  );
  return <input placeholder="Enter your name:" {...props} ref={inputRef} />;
});

export default MyInput;
~~~

Обратите внимание, что в приведенном выше коде файл `ref` больше не перенаправляется в `<input>`.

Теперь, если родительский компонент получит ссылку на `MyInput`, он сможет вызывать для него методы `focus` и `scrollIntoView`. Однако у него не будет полного доступа к базовому узлу DOM.

> Метод `Element.scrollIntoView()` интерфейса `Element` прокручивает контейнер родителя элемента так, чтобы элемент, на котором был вызван `scrollIntoView()`, стал виден пользователю.

Методы, которые вы предоставляете через императивный дескриптор, не обязательно точно соответствуют методам DOM. Например, этот компонент `Post` предоставляет метод `scrollAndFocusAddComment` через императивный дескриптор. Это позволяет родительской странице прокручивать список комментариев и фокусировать поле ввода при нажатии кнопки:

App.tsx
~~~
import { useRef } from "react";
import Post from "./Post";

interface IRef {
  scrollAndFocusAddComment: () => void;
}

export default function Page() {
  const postRef = useRef<IRef>(null);

  function handleClick() {
    postRef.current!.scrollAndFocusAddComment();
  }

  return (
    <>
      <button onClick={handleClick}>Write a comment</button>
      <Post ref={postRef} />
    </>
  );
}
~~~

Post.tsx
~~~
import { forwardRef, useRef, useImperativeHandle } from "react";
import CommentList from "./CommentList";
import AddComment from "./AddComment";

interface IRef {
  scrollToBottom: () => void;
}

const Post = forwardRef((props, ref) => {
  const commentsRef = useRef<IRef>(null);
  const addCommentRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        scrollAndFocusAddComment() {
          commentsRef.current!.scrollToBottom();
          addCommentRef.current!.focus();
        },
      };
    },
    []
  );

  return (
    <>
      <article>
        <p>Welcome to my blog!</p>
      </article>
      <CommentList ref={commentsRef} />
      <AddComment ref={addCommentRef} />
    </>
  );
});

export default Post;
~~~

CommentList.tsx
~~~
import { forwardRef, useRef, useImperativeHandle } from "react";

const CommentList = forwardRef(function CommentList(props, ref) {
  const divRef = useRef<HTMLDivElement>(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        scrollToBottom() {
          const node = divRef.current;
          node!.scrollTop = node!.scrollHeight;
        },
      };
    },
    []
  );

  let comments = [];
  for (let i = 0; i < 50; i++) {
    comments.push(<p key={i}>Comment #{i}</p>);
  }

  return (
    <div className="CommentList" ref={divRef}>
      {comments}
    </div>
  );
});

export default CommentList;
~~~

AddComment.tsx
~~~
import { forwardRef } from "react";

const AddComment = forwardRef<HTMLInputElement>(function AddComment(
  props,
  ref
) {
  return <input placeholder="Add comment..." ref={ref} />;
});

export default AddComment;
~~~

### Ловушка

Не злоупотребляйте `refs`. Вы должны использовать `refs` только для императивного поведения, которое вы не можете выразить как свойства: например, прокрутка до узла, фокусировка на узле, запуск анимации, выделение текста и т. д.

Если вы можете выразить что-то в качестве пропса, вы не должны использовать `ref`. Например, вместо того, чтобы предоставлять императивный дескриптор, такой как `{ open, close }`, из модального компонента, лучше использовать `isOpen` в качестве пропса, такого как `<Modal isOpen={isOpen} />`.

# ?HOC

_Higher-Order Component_ (компонент высшего порядка) — это функция, которая принимает компонент и возвращает новый, измененный компонент.

Если обычный компонент преобразует пропсы в UI, то компонент высшего порядка преобразует компонент в другой компонент. HOC полностью контролирует процесс создания компонента.

HOC могут легко позволить вам уменьшить количество повторяющейся логики в вашем приложении. Рассмотрим пример `CommentList`, который получает список комментариев из внешнего источника данных и отображает их. И `BlogPost`, который повторяет уже знакомый нам шаблон:

App.tsx
~~~
class App extends React.Component {
  render() {
    return (
      <>
        <CommentList />
        <BlogPost />
      </>
    );
  }
}
~~~

DataSource.tsx
~~~
export interface IDataSource {
  getComments: () => string[];
  getBlogPost: () => string;
  addChangeListener: () => void;
  removeChangeListener: () => void;
}

class DataSource {
  static getComments() {
    return ["cool", "great"];
  }

  static getBlogPost() {
    return "post";
  }

  static addChangeListener() {
    console.log("add");
  }

  static removeChangeListener() {
    console.log("remove");
  }
}
~~~

CommentList.tsx
~~~
class CommentList extends React.Component<object, { comments: string[] }> {
  constructor(props: object) {
    super(props);
    this.state = {
      comments: DataSource.getComments(),
    };
  }

  componentDidMount() {
    DataSource.addChangeListener();
  }

  componentWillUnmount() {
    DataSource.removeChangeListener();
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <div key={comment}>{comment}</div>
        ))}
      </div>
    );
  }
}
~~~

BlogPost.tsx
~~~
class BlogPost extends React.Component<object, { blogPost: string }> {
  constructor(props: object) {
    super(props);
    this.state = {
      blogPost: DataSource.getBlogPost(),
    };
  }

  componentDidMount() {
    DataSource.addChangeListener();
  }

  componentWillUnmount() {
    DataSource.removeChangeListener();
  }

  render() {
    return <div>{this.state.blogPost}</div>;
  }
}
~~~

Разница между `CommentList` и `BlogPost` в том, что они вызывают разные методы `DataSource` и рендерят разный вывод. Однако в большинстве своём они похожи:

* Оба компонента подписываются на оповещения от `DataSource` при монтировании.
* Оба отписываются от `DataSource` при размонтировании.

Было бы здорово абстрагировать эту функциональность.

Давайте реализуем функцию `withSubscription` — она будет создавать компоненты и подписывать их на обновления `DataSource` (наподобие `CommentList` и `BlogPost`). Функция будет принимать оборачиваемый компонент и через пропсы передавать ему новые данные:

withSubscription.tsx
~~~
function withSubscription(
  WrappedComponent: ComponentType<{ data: string | string[] }>,
  selectData: (DataSource: IDataSource) => string | string[]
) {
  return class extends React.Component<object, { data: string | string[] }> {
    constructor(props: object) {
      super(props);
      this.state = {
        data: selectData(DataSource),
      };
    }

    componentDidMount() {
      DataSource.addChangeListener();
    }

    componentWillUnmount() {
      DataSource.removeChangeListener();
    }

    render() {
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
~~~

CommentList.tsx
~~~
class CommentList extends React.Component<{ data: string | string[] }> {
  render() {
    return (
      <>
        {(this.props.data as string[]).map((comment) => (
          <div key={comment}>{comment}</div>
        ))}
      </>
    );
  }
}

export default withSubscription(CommentList, (DataSource: IDataSource) =>
  DataSource.getComments()
);
~~~

BlogPost.tsx
~~~
class BlogPost extends React.Component<{ data: string | string[] }> {
  render() {
    return <div>{this.props.data}</div>;
  }
}

export default withSubscription(BlogPost, (DataSource: IDataSource) =>
  DataSource.getBlogPost()
);
~~~

HOC оборачивает оригинальный компонент в контейнер посредством композиции. HOC является чистой функцией без побочных эффектов.

### HOC vs Custom hook

* HOC может отображать JSX, а хуки — нет. 
* HOC — это независимый от React шаблон, но хуки являются частью React.
* Мы можем создавать HOC как функции, но React Hooks не компонуемы.
* HOC добавляет еще один компонент в иерархию компонентов (мы можем проверить это в DevTools).

Тот же пример, только на функциональных компоненах и HOC как функция:

App.tsx
~~~
function App() {
  return (
    <>
      <CommentList />
      <BlogPost />
    </>
  );
}
~~~

DataSource.tsx
~~~
export interface IDataSource {
  getComments: () => string[];
  getBlogPost: () => string;
  addChangeListener: () => void;
  removeChangeListener: () => void;
}

class DataSource {
  static getComments() {
    return ["cool", "great"];
  }

  static getBlogPost() {
    return "post";
  }

  static addChangeListener() {
    console.log("add");
  }

  static removeChangeListener() {
    console.log("remove");
  }
}
~~~

CommentList.tsx
~~~
function CommentList(props: { data: string | string[] }) {
  return (
    <>
      {(props.data as string[]).map((comment) => (
        <div key={comment}>{comment}</div>
      ))}
    </>
  );
}

export default withSubscription(CommentList, (DataSource: IDataSource) =>
  DataSource.getComments()
);
~~~

BlogPost.tsx
~~~
function BlogPost(props: { data: string | string[] }) {
  return <div>{props.data}</div>;
}

export default withSubscription(BlogPost, (DataSource: IDataSource) =>
  DataSource.getBlogPost()
);
~~~

withSubscription.tsx
~~~
function withSubscription(
  WrappedComponent: ComponentType<{ data: string | string[] }>,
  selectData: (DataSource: IDataSource) => string | string[]
) {
  return (props: {}) => {
    const data = selectData(DataSource);

    React.useEffect(() => {
      DataSource.addChangeListener();
      return () => {
        DataSource.removeChangeListener();
      };
    }, []);

    return <WrappedComponent data={data} {...props} />;
  };
}
~~~

Соглашения:

* Пропсы, которые напрямую не связаны с функциональностью HOC, должны передаваться без изменений оборачиваемому компоненту. 
* Для более лёгкой отладки вы можете задать имя, которое подскажет, что определённый компонент был создан с помощью HOC.

~~~
function withSubscription(
  WrappedComponent: ComponentType<{ data: string | string[] }>,
  selectData: (DataSource: IDataSource) => string | string[]
) {
  function Component(props: {}) {
    const data = selectData(DataSource);
    React.useEffect(() => {
      DataSource.addChangeListener();
      return () => {
        DataSource.removeChangeListener();
      };
    }, []);
    return <WrappedComponent data={data} {...props} />;
  }
  Component.displayName = "WithSubscription";
  return Component;
}
~~~

### Предостережения

* Не используйте HOC внутри `render`-метода. При необходимости (в редких случаях) можно динамически применять HOC в методах жизненного цикла или конструкторе компонента.
* Когда мы применяем HOC, то заворачиваем оригинальный компонент в контейнер. Поэтому у нового компонента не будет статических методов оригинального компонента. Скопируйте недостающие методы в контейнер или воспользуйтесь `hoist-non-react-statics`, чтобы автоматически скопировать не связанные с React статические методы. Другое возможное решение — экспортировать статические методы отдельно от компонента.
* Рефы не передаются. Вы можете решить эту проблему с помощью API-метода `React.forwardRef`.

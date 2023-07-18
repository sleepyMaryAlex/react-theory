# ?Reducers

`Reducers` определяют, как состояние приложения изменяется в ответ на экшены, отправленные в стор. Помните, что экшены только описывают, что произошло, но не описывают, как изменяется состояние приложения.

`Reducer` — это чистая функция, которая принимает предыдущее состояние и экшен (`state` и `action`) и возвращает следующее состояние (новую версию предыдущего).

Очень важно, чтобы редьюсеры оставались чистыми функциями. Получая аргументы одного типа, редьюсер должен вычислять новую версию состояния и возвращать ее. Никаких сюрпризов. Никаких сайд-эффектов. Никаких обращений к стороннему API. Никаких изменений (mutations). Только вычисление новой версии состояния.

~~~
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter,
      });
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false,
          },
        ],
      });
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if (index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed,
            });
          }
          return todo;
        }),
      });
    default:
      return state;
  }
}
~~~

Есть ли способ облегчить понимание? Кажется, что `todos` и `visibilityFilter` обновляются совершенно независимо.

~~~

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false,
        },
      ];
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed,
          });
        }
        return todo;
      });
    default:
      return state;
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter;
    default:
      return state;
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(
      state.visibilityFilter,
      action
    ),
    todos: todos(state.todos, action),
  };
}
~~~

Это называется композицией редьюсеров и является фундаментальным шаблоном построения Redux-приложений.

Каждый из этих дочерних редьюсеров управляет только какой-то одной частью глобального состояния. Параметр `state` разный для каждого отдельного дочернего редьюсера и соответствует той части глобального состояния, которой управляет этот дочерний редьюсер.

Наконец, Redux предоставляет утилиту, называемую `combineReducers()`, которая реализует точно такой же логический шаблон, который мы только что реализовали в todoApp. С ее помощью мы можем переписать `todoApp` следующим образом:

~~~

import { combineReducers } from 'redux';
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters,
} from './actions';
const { SHOW_ALL } = VisibilityFilters;

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter;
    default:
      return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false,
        },
      ];
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed,
          });
        }
        return todo;
      });
    default:
      return state;
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos,
});

export default todoApp;
~~~

Все, что делает `combineReducers()` — это генерирует функцию, которая вызывает ваши редьюсеры c частью глобального состояния, которая выбирается в соответствии с их ключами, и затем снова собирает результаты всех вызовов в один объект.

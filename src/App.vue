<script>
import todos from './data/todos';
import StatusFilter from './components/StatusFilter.vue';
import TodoItem from './components/TodoItem.vue';

function getFromLocalStorage(key, defaultValue) {
  try {
    return JSON.parse(localStorage.getItem(key));
  } catch (e) {
    return defaultValue;
  }
}

export default {
  name: 'App',
  components: {
    StatusFilter,
    TodoItem,
  },
  data() {
    return {
      todos: getFromLocalStorage('todos', []),
      title: '',
      status: 'all',
    };
  },

  watch: {
    todos: {
      handler() {
        console.log('Todos changed!', this.todos);
        localStorage.setItem('todos', JSON.stringify(this.todos));
      },
      deep: true,
    },
  },
  computed: {
    activeTodos() {
      return this.todos.filter((todo) => !todo.completed);
    },
    completedTodos() {
      return this.todos.filter((todo) => todo.completed);
    },
    visibleTodos() {
      if (this.status === 'all') {
        return this.todos;
      }

      if (this.status === 'active') {
        return this.activeTodos;
      }

      if (this.status === 'completed') {
        return this.completedTodos;
      }

      throw new Error(`Unknown status filter: ${this.status}`);
    },
  },
  methods: {
    addTodo() {
      if (!this.title) {
        return;
      }

      const newTodo = {
        id: Date.now(),
        title: this.title,
        completed: false,
      };

      this.todos.push(newTodo);
      this.title = '';
    },
  },
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
        ></button>

        <form @submit.prevent="addTodo">
          <input
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </header>

      <TransitionGroup name="list" tag="section" class="todoapp__main">
        <TodoItem
          v-for="(todo, index) of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @update="Object.assign(todo, $event)"
          @delete="todos.splice(todos.indexOf(todo), 1)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer">
        <span class="todo-count"> {{ activeTodos.length }} items left </span>

        <StatusFilter v-model="status" />

        <button class="todoapp__clear-completed" v-if="activeTodos.length > 0">
          Clear completed
        </button>
      </footer>
    </div>

    <article class="message is-danger message--hidden">
      <div class="message-header">
        <p>Error</p>
        <button class="delete"></button>
      </div>

      <div class="message-body">Unable to add a Todo</div>
    </article>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  max-height: 0;
  transform: translateY(0);
}
</style>

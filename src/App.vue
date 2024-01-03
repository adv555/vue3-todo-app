<script>
// import todos from './data/todos';
import StatusFilter from './components/StatusFilter.vue';
import TodoItem from './components/TodoItem.vue';
import Message from './components/Message.vue';
import * as todosApi from './api/todos';

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
    Message,
  },
  data() {
    return {
      // todos: getFromLocalStorage('todos', []),
      todos: [],
      title: '',
      status: 'all',
      errorMessage: '',
    };
  },

  // watch: {
  //   todos: {
  //     handler() {
  //       console.log('Todos changed!', this.todos);
  //       localStorage.setItem('todos', JSON.stringify(this.todos));
  //     },
  //     deep: true,
  //   },
  // },
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
  mounted() {
    console.log('App mounted!', this.todos);
    todosApi
      .getTodos()
      .then(({ data }) => {
        console.log('Todos from API', data);
        this.todos = data;
      })
      .catch(() => {
        this.$refs.errorMessage.show('Failed to load todos');
      });
  },
  methods: {
    handleSubmit() {
      todosApi.createTodo(this.title).then(({ data }) => {
        this.todos = [...this.todos, data];
        this.title = '';
      });
    },
    updateTodo({ id, title, completed }) {
      todosApi.updateTodo({ id, title, completed }).then(({ data }) => {
        this.todos = this.todos.map((todo) => (todo.id !== id ? todo : data));
      });
    },
    deleteTodo(todoId) {
      todosApi.deleteTodo(todoId).then(() => {
        this.todos = this.todos.filter((todo) => todo.id !== todoId);
      });
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

        <form @submit.prevent="handleSubmit">
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
          @update="updateTodo"
          @delete="deleteTodo(todo.id)"
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

    <Message class="is-warning" ref="errorMessage">
      <template #default="{ text }">
        <p>{{ text }}</p>
      </template>

      <template #header>
        <p>Server Error</p>
      </template>
    </Message>
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

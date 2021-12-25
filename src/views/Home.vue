<template>
  <section class="todoapp" v-cloak>
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        autofocus
        autocomplete="off"
        placeholder="What needs to be done?"
        v-model.trim="newTodo"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main" v-show="todos.length">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="allDone"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          class="todo"
          v-for="todo in filteredTodos"
          :key="todo.id"
          :class="{
            completed: todo.completed,
            editing: todo === editedTodo
          }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" />
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input
            class="edit"
            type="text"
            v-model="todo.title"
            v-todo-focus="todo == editedTodo"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length">
      <span class="todo-count">
        <strong v-text="remaining"></strong>
        {{ $options.filters.pluralize("item", remaining) }} left
      </span>
      <ul class="filters">
        <li>
          <a href="#/all" :class="{ selected: visibility == 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: visibility == 'active' }"
            >Active</a
          >
        </li>
        <li>
          <a href="#/completed" :class="{ selected: visibility == 'completed' }"
            >Completed</a
          >
        </li>
      </ul>
      <button
        class="clear-completed"
        @click="removeCompleted"
        v-show="todos.length > remaining"
      >
        Clear completed
      </button>
    </footer>
  </section>
</template>

<script>

export default {
  name: 'Home',
  directives: {
    'todo-focus': function (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  },
  filters: {
    pluralize (word, num) {
      return num === 1 ? word : word + 's'
    }
  },
  data () {
    return {
      newTodo: '',
      todos: [],
      editedTodo: null,
      allDone: false
    }
  },
  computed: {
    filteredTodos () {
      const filter = this.visibility
      switch (filter) {
        case 'active':
          return this.activeTodos
        case 'completed':
          return this.completedTodos
        default:
          return this.todos
      }
    },
    activeTodos () {
      return this.todos.filter(todo => !todo.completed)
    },
    completedTodos () {
      return this.todos.filter(todo => todo.completed)
    },
    remaining () {
      return this.activeTodos.length
    },
    visibility () {
      const filter = this.$route.hash.replace(/#\/?/, '')
      return filter || 'all'
    }
  },
  methods: {
    addTodo () {
      const dateTime = Date.now()
      const timestamp = Math.floor(dateTime / 1000)
      this.todos.splice(this.todos.length, 0, {
        id: timestamp,
        title: this.newTodo,
        completed: false
      })
      this.newTodo = ''
    },
    editTodo (todo) {
      this.editedTodo = todo
    },
    removeTodo ({ id }) {
      const index = this.todos.findIndex(todo => todo.id === id)
      this.todos.splice(index, 1)
    },
    doneEdit ({ id }) {
      const index = this.todos.findIndex(todo => todo.id === id)
      this.todos.splice(index, 1, this.editedTodo)
      this.cancelEdit()
    },
    cancelEdit () {
      this.editedTodo = null
    },
    removeCompleted () {
      this.completedTodos.forEach(todo => this.removeTodo(todo))
    }
  }
}
</script>

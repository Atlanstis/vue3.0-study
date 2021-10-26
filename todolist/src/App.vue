<template>
  <section id="app" class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autocomplete="off"
        autofocus
        v-model="input"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main">
      <input id="toggle-all" class="toggle-all" type="checkbox" />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li v-for="(todo, index) of todos" :key="index">
          <div class="view">
            <input class="toggle" type="checkbox" />
            <label>{{ todo.text }}</label>
            <button class="destroy"></button>
          </div>
          <input class="edit" type="text" />
        </li>
      </ul>
    </section>
    <footer class="footer">
      <span class="todo-count">
        <strong>10</strong>
        items left
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button class="clear-completed">
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
  </footer>
</template>

<script>
import './assets/index.css'
import { ref } from 'vue'

// 添加待办事项
const useAdd = (todos) => {
  const input = ref('')

  const addTodo = () => {
    const text = input.value && input.value.trim()
    if (text.length) {
      todos.value.unshift({
        text,
        completed: false
      })
    }
    input.value = ''
  }
  return {
    input,
    addTodo
  }
}

export default {
  name: 'Todos',

  setup() {
    const todos = ref([])

    return {
      todos,
      ...useAdd(todos)
    }
  }
}
</script>

<style></style>

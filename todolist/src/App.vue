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
        <li
          v-for="(todo, index) of todos"
          :key="index"
          :class="{ editing: todo._editing }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" />
            <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="removeTodo(todo, index)"></button>
          </div>
          <input
            v-model="todo.text"
            v-editing-focus="todo._editing"
            class="edit"
            type="text"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
            @blur="doneEdit(todo)"
          />
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
        completed: false,
        _editing: false
      })
    }
    input.value = ''
  }
  return {
    input,
    addTodo
  }
}

// 删除待办事项
const useRemove = (todos) => {
  const removeTodo = (index) => {
    todos.value.splice(index, 1)
  }
  return { removeTodo }
}

// 编辑待办事项
const useEdit = (removeTodo) => {
  let beforeEditText = ''

  // 开始编辑
  const editTodo = (todo) => {
    beforeEditText = todo.text
    todo._editing = true
  }

  // 编辑完成
  const doneEdit = (todo, index) => {
    if (todo._editing) {
      todo._editing = false
      todo.text = todo.text.trim()
      if (!todo.text) {
        removeTodo(index)
      }
    }
  }
  // 取消编辑
  const cancelEdit = (todo) => {
    if (todo._editing) {
      todo.text = beforeEditText
      todo._editing = false
    }
  }

  return {
    editTodo,
    doneEdit,
    cancelEdit
  }
}

export default {
  name: 'Todos',

  setup() {
    const todos = ref([])
    const { removeTodo } = useRemove(todos)
    return {
      todos,
      ...useAdd(todos),
      removeTodo,
      ...useEdit(removeTodo)
    }
  },

  directives: {
    editingFocus: (el, binding) => {
      binding.value && el.focus()
    }
  }
}
</script>

<style></style>

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
    <section v-show="count" class="main">
      <input
        v-model="allDone"
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          v-for="(todo, index) of filteredTodos"
          :key="todo"
          :class="{ editing: todo._editing, completed: todo.completed }"
        >
          <div class="view">
            <input v-model="todo.completed" class="toggle" type="checkbox" />
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
    <footer v-show="count" class="footer">
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong>
        {{ remainingCount > 1 ? 'items' : 'item' }} left
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button
        v-show="count > remainingCount"
        @click="removeComplted"
        class="clear-completed"
      >
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
import { computed, onMounted, onUnmounted, ref } from 'vue'

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

  const removeComplted = () => {
    todos.value = todos.value.filter((todo) => !todo.completed)
  }

  return { removeTodo, removeComplted }
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

// 切换待办事项完成状态
const useFilter = (todos) => {
  const allDone = computed({
    get: () => {
      return !todos.value.filter((todo) => {
        return !todo.completed
      }).length
    },
    set: (value) => {
      todos.value.forEach((todo) => {
        todo.completed = value
      })
    }
  })

  const type = ref('all')
  const filteredTodos = computed(() => filter[type.value](todos.value))
  const remainingCount = computed(() => filter.active(todos.value).length)
  const count = computed(() => todos.value.length)

  const filter = {
    all: (list) => list,
    active: (list) => list.filter((todo) => !todo.completed),
    completed: (list) => list.filter((todo) => todo.completed)
  }

  // 监听 hash 变化，切换显示数据
  const onHashChange = () => {
    const hash = window.location.hash.replace('#/', '')
    if (filter[hash]) {
      type.value = hash
    } else {
      type.value = 'all'
      window.location.hash = ''
    }
  }

  onMounted(() => {
    window.addEventListener('hashchange', onHashChange)
    onHashChange()
  })

  onUnmounted(() => {
    window.removeEventListener('hashchange', onHashChange)
  })

  return {
    allDone,
    filteredTodos,
    remainingCount,
    count
  }
}

export default {
  name: 'Todos',

  setup() {
    const todos = ref([])
    const { removeTodo, removeComplted } = useRemove(todos)
    return {
      todos,
      ...useAdd(todos),
      removeTodo,
      removeComplted,
      ...useEdit(removeTodo),
      ...useFilter(todos)
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

// TodoList.vue
<template>
  <div class="todo-container">
    <h1 class="text-2xl font-bold mb-6">待办事项清单</h1>
    
    <!-- 添加待办事项表单 -->
    <div class="add-todo-form mb-6">
      <input
        v-model="newTodo"
        @keyup.enter="addTodo"
        placeholder="请输入新的待办事项"
        class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:border-blue-500"
      />
      <button
        @click="addTodo"
        class="mt-2 w-full bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition-colors"
      >
        添加
      </button>
    </div>

    <!-- 待办事项列表 -->
    <div class="todo-list">
      <TransitionGroup name="list">
        <div
          v-for="todo in todos"
          :key="todo._id"
          class="todo-item mb-3 p-4 bg-white rounded-lg shadow flex items-center"
        >
          <input
            type="checkbox"
            :checked="todo.completed"
            @change="toggleTodo(todo)"
            class="mr-4"
          />
          <div class="flex-grow">
            <span
              :class="{ 'line-through text-gray-500': todo.completed }"
              class="text-lg"
            >
              {{ todo.title }}
            </span>
            <div class="text-sm text-gray-500">
              {{ formatDate(todo.created_at) }}
            </div>
          </div>
          <button
            @click="deleteTodo(todo)"
            class="text-red-500 hover:text-red-700 transition-colors"
          >
            删除
          </button>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
let { VITE_BASE_URL } = import.meta.env
console.log('VITE_BASE_URL',VITE_BASE_URL)

const API_URL = `${VITE_BASE_URL}/api`
const todos = ref([])
const newTodo = ref('')

// 格式化日期
const formatDate = (date) => {
  return new Date(date).toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

// 获取所有待办事项
const fetchTodos = async () => {
  try {
    const response = await axios.get(`${API_URL}/todos`)
    todos.value = response.data
  } catch (error) {
    console.error('获取待办事项失败:', error)
  }
}

// 添加待办事项
const addTodo = async () => {
  if (!newTodo.value.trim()) return
  
  try {
    const response = await axios.post(`${API_URL}/todos`, {
      title: newTodo.value
    })
    todos.value.unshift(response.data)
    newTodo.value = ''
  } catch (error) {
    console.error('添加待办事项失败:', error)
  }
}

// 切换待办事项状态
const toggleTodo = async (todo) => {
  try {
    const response = await axios.put(`${API_URL}/todos/${todo._id}`, {
      completed: !todo.completed
    })
    const index = todos.value.findIndex(t => t._id === todo._id)
    todos.value[index] = response.data
  } catch (error) {
    console.error('更新待办事项失败:', error)
  }
}

// 删除待办事项
const deleteTodo = async (todo) => {
  try {
    await axios.delete(`${API_URL}/todos/${todo._id}`)
    todos.value = todos.value.filter(t => t._id !== todo._id)
  } catch (error) {
    console.error('删除待办事项失败:', error)
  }
}

// 组件挂载时获取待办事项
onMounted(fetchTodos)
</script>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>
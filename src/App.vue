<script setup>
import { computed, ref, onMounted, TransitionGroup } from "vue";
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from "./components/TodoItem.vue";
import * as todoApi from "./api/todos";
const todos = ref([]);

onMounted(async () => {
  todos.value = await todoApi.getTodos();
});

const activeTodos = computed(() =>
  todos.value.filter((todo) => !todo.completed)
);

const title = ref("");

const addTodo = async () => {
  if (!title.value) return;

  const newTodo = await todoApi.createTodo(title.value);

  todos.value.push(newTodo);
  title.value = "";
};

const errorMessage = ref("");
const status = ref("all");

const visibleTodos = computed(() => {
  if (status.value === "active") {
    return activeTodos.value;
  }

  if (status.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  }

  return todos.value;
});

const updateTodo = async ({ id, title, completed }) => {
  const updatedTodo = await todoApi.updateTodo({ id, title, completed });
  const currentTodo = todos.value.find((todo) => todo.id === id);
  Object.assign(currentTodo, updatedTodo);
};
const deleteTodo = async (todoId) => {
  await todoApi.deleteTodo(todoId);
  todos.value = todos.value.filter((todo) => todoId !== todo.id)
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ todos.length }}</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
          v-if="todos.length > 0"
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
        ></button>

        <form @submit.prevent="addTodo">
          <input
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </header>

      <TransitionGroup
        tag="section"
        name="todolist"
        class="todoapp__main"
        v-if="todos.length > 0"
      >
        <TodoItem
          v-for="todo of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @delete="deleteTodo(todo.id)"
          @update="updateTodo($event)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer" v-if="todos.length > 0">
        <span class="todo-count">{{ activeTodos.length }} items left</span>

        <StatusFilter v-model="status" />

        <button
          class="todoapp__clear-completed"
          :disabled="todos.length === activeTodos.length"
          @click="todos = activeTodos"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <div
      v-if="errorMessage"
      class="notification is-danger is-light has-text-weight-normal hidden"
    >
      <button class="delete" @click="errorMessage = ''"></button>
      Unable to load todos<br />
      Title should not be empty<br />
      Unable to add a todo<br />
      Unable to delete a todo<br />
      Unable to update a todo<br />
    </div>
  </div>
</template>

<style scoped>
.todolist-enter-active,
.todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.todolist-enter-from,
.todolist-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>

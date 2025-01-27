<script setup>
import { computed, ref, watch, onBeforeMount } from "vue";
import StatusFilter from "./components/StatusFilter.vue";

const todos = ref([]);

onBeforeMount(() => {
  try {
    todos.value = JSON.parse(localStorage.getItem('todos'));
  } catch (error) {}

  if (!Array.isArray(todos.value)) {
    todos.value = [];
  }
});

watch(
  todos,
  (newTodos) => {
    localStorage.setItem("todos", JSON.stringify(newTodos));
  },
  { deep: true }
);
const activeTodos = computed(() =>
  todos.value.filter((todo) => !todo.completed)
);

const title = ref("");

function addTodo() {
  if (!title.value) {
    errorMessage.value = "Title should not be empty";

    return;
  }

  todos.value.push({
    id: Date.now(),
    title: title.value,
    completed: false,
  });

  title.value = "";
}

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

      <section class="todoapp__main" v-if="todos.length > 0">
        <div
          v-for="(todo, i) of visibleTodos"
          :key="todo.id"
          class="todo"
          :class="{ completed: todo.completed }"
        >
          <label class="todo__status-label">
            <input
              type="checkbox"
              class="todo__status"
              v-model="todo.completed"
            />
          </label>

          <form v-if="false">
            <input
              class="todo__title-field"
              placeholder="Empty todo will be deleted"
            />
          </form>

          <template v-else>
            <span class="todo__title">{{ todo.title }}</span>
            <button class="todo__remove" @click="todos.splice(i, 1)">×</button>
          </template>

          <div class="modal overlay" :class="{ 'is-active': false }">
            <div class="modal-background has-background-white-ter"></div>
            <div class="loader"></div>
          </div>
        </div>
      </section>

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

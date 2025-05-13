<script setup>
import { ref, computed, watch, onBeforeMount, TransitionGroup } from "vue";
import originalTodos from "./data/todos";
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from "./components/TodoItem.vue";

const todos = ref([]);

onBeforeMount(() => {
  try {
    todos.value = JSON.parse(localStorage.getItem("todos"));
  } catch (error) {}

  if (!Array.isArray(todos.value)) {
    todos.value = [];
  }
});

const title = ref("");
const errorMessage = ref("");
const activeTodos = computed(() =>
  todos.value.filter((todo) => !todo.completed)
);
const status = ref("all");

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

const visibleTodos = computed(() => {
  if (status.value === "active") {
    return activeTodos.value;
  }

  if (status.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  }

  return todos.value;
});

watch(
  todos,
  (newTodos) => {
    localStorage.setItem("todos", JSON.stringify(newTodos));
  },
  { deep: true }
);
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ todos.length }}</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <!-- this button should have `active` class only if all todos are
        completed -->
        <button
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
          data-cy="ToggleAllButton"
          v-if="todos.length > 0"
        ></button>

        <!-- Add a todo on form submit -->
        <form @submit.prevent="addTodo">
          <input
            data-cy="NewTodoField"
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
            @input="errorMessage = ''"
          />
        </form>
      </header>

      <TransitionGroup
        tag="section"
        name="todolist"
        class="todoapp__main"
        data-cy="TodoList"
        v-if="todos.length > 0"
      >
        <!-- This is a todos list -->
        <TodoItem
          v-for="todo of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @delete="todos.splice(todos.indexOf(todo), 1)"
          @update="Object.assign(todo, $event)"
        />
      </TransitionGroup>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer" data-cy="Footer">
        <span class="todo-count" data-cy="TodosCounter">
          {{ activeTodos.length }} items left
        </span>

        <!-- Active link should have the 'selected' class -->
        <StatusFilter v-model="status" />

        <!-- this button should be disabled if there are no completed todos -->
        <button
          class="todoapp__clear-completed"
          data-cy="ClearCompletedButton"
          :disabled="todos.length === activeTodos.length"
          @click="todos = activeTodos"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <!-- DON'T use conditional rendering to hide the notification -->
    <!-- Add the
    'hidden' class to hide the message smoothly -->
    <div
      v-if="errorMessage"
      class="notification is-danger is-light has-text-weight-normal"
    >
      <button class="delete" @click="errorMessage = ''"></button>

      {{ errorMessage }}
    </div>

    <div
      data-cy="ErrorNotification"
      class="notification is-danger is-light has-text-weight-normal"
    >
      <button data-cy="HideErrorButton" class="delete"></button>
      <!-- show only one message at a time -->
      Unable to load todos
      <br />
      Title should not be empty
      <br />
      Unable to add a todo
      <br />
      Unable to delete a todo
      <br />
      Unable to update a todo
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

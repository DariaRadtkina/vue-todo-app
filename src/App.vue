<script setup>
import { ref, computed, onMounted, TransitionGroup } from "vue";
import originalTodos from "./data/todos";
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from "./components/TodoItem.vue";
import * as todoApi from "./api/todos";
import Message from "./components/Message.vue";

const todos = ref([]);

onMounted(async () => {
  try {
    todos.value = await todoApi.getTodos();
  } catch (error) {
    errorMessage.value.show("Unable to load todos");
  }
});

const title = ref("");
const errorMessage = ref(null);
const activeTodos = computed(() =>
  todos.value.filter((todo) => !todo.completed)
);
const status = ref("all");

const addTodo = async () => {
  if (!title.value) {
    errorMessage.value.show("Title should not be empty");
    return;
  }

  try {
    const newTodo = await todoApi.createTodo(title.value);

    todos.value.push(newTodo);
    title.value = "";
  } catch (error) {
    errorMessage.value.show("Unable to add a todo");
  }
};

const updateTodo = async ({ id, title, completed }) => {
  try {
    const updatedTodo = await todoApi.updateTodo({ id, title, completed });
    const currentTodo = todos.value.find((todo) => todo.id === id);

    Object.assign(currentTodo, updatedTodo);
  } catch (error) {
    errorMessage.value.show("Unable to update a todo");
  }
};

const deleteTodo = async (todoId) => {
  try {
    await todoApi.deleteTodo(todoId);
    todos.value = todos.value.filter((todo) => todoId !== todo.id);
  } catch (error) {
    errorMessage.value.show("Unable to delete a todo");
  }
};

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
          @delete="deleteTodo(todo.id)"
          @update="updateTodo($event)"
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

    <Message class="is-danger" ref="errorMessage">
      <template #header>
        <p>Server Error</p>
      </template>

      <template #default="{ text }">
        <p>{{ text }}</p>
      </template>
    </Message>
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

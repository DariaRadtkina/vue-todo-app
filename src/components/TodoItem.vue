<script setup>
import { defineProps, defineEmits, nextTick, ref } from "vue";

const props = defineProps(["todo"]);
const emit = defineEmits(["delete", "update"]);

const editing = ref(false);
const titleField = ref(null);
const newTitle = ref(props.todo.title);

const startEditing = async () => {
  newTitle.value = props.todo.title;
  editing.value = true;

  await nextTick();

  if (titleField.value) {
    titleField.value.focus();
  }
};

const rename = () => {
  if (!editing.value) return;

  editing.value = false;

  if (newTitle.value === props.todo.title) {
    return;
  }

  if (!newTitle.value) {
    emit('delete');
    return;
  }

  emit('update', { ...props.todo, title: newTitle.value });
};


</script>

<template>
  <div data-cy="Todo" class="todo" :class="{ completed: props.todo.completed }">
    <label class="todo__status-label">
      <input
        data-cy="TodoStatus"
        type="checkbox"
        class="todo__status"
        :checked="props.todo.completed"
        @change="
          emit('update', { ...props.todo, completed: !props.todo.completed })
        "
      />
    </label>

    <!-- show when todo is being edited -->
    <form v-if="editing" @submit.prevent="rename">
      <input
        data-cy="TodoTitleField"
        class="todo__title-field"
        @keyup.escape="editing = false"
        placeholder="Empty todo will be deleted"
        ref="titleField"
        v-model.trim="newTitle"
        @blur="rename"
      />
    </form>
    <!-- show when todo is being edited -->
    <template v-else>
      <span data-cy="TodoTitle" class="todo__title" @dblclick="startEditing">
        {{ props.todo.title }}
      </span>
      <button class="todo__remove" data-cy="TodoDelete" @click="emit('delete')">
        ×
      </button>
    </template>

    <!-- add `is-active` class when todo being processed -->
    <div
      data-cy="TodoLoader"
      class="modal overlay"
      :class="{ 'is-active': false }"
    >
      <div class="modal-background has-background-white-ter"></div>
      <div class="loader"></div>
    </div>
  </div>
</template>

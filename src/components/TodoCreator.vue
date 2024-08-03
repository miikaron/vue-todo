<script setup>
import { reactive, watch } from "vue";

const props = defineProps({
  todoList: {
    type: Array,
    default: () => [],
  },
});

const emit = defineEmits(["create-todo", "update-filter-text"]);

const todo = reactive({
  todo: "",
  invalid: false,
  errMsg: "",
  filterMsg: "",
  buttonText: "Create",
});

watch(
  () => todo.todo,
  (newVal) => {
    filterTodoList(newVal);
    clearErrorMsg(); // Clear error message when input changes
    emit("update-filter-text", newVal); // Emit filter text to parent
  }
);

const filterTodoList = (newVal) => {
  const existingTodo = props.todoList.find((item) => item.todo === newVal);
  if (newVal === "") {
    todo.filterMsg = "";
    todo.buttonText = "Create";
  } else if (!existingTodo) {
    todo.filterMsg = "This todo does not exist yet. Add it!";
    todo.buttonText = "Add";
  } else {
    todo.filterMsg = "";
    todo.buttonText = "Create";
  }
};

const clearErrorMsg = () => {
  if (todo.invalid) {
    todo.invalid = false;
    todo.errMsg = "";
  }
};

const createTodo = () => {
  const existingTodo = props.todoList.find((item) => item.todo === todo.todo);
  if (todo.todo !== "" && !existingTodo) {
    emit("create-todo", todo.todo);
    todo.todo = "";
    todo.filterMsg = "";
    todo.buttonText = "Create";
  } else if (existingTodo) {
    todo.invalid = true;
    todo.errMsg = "This todo already exists!";
  } else {
    todo.invalid = true;
    todo.errMsg = "Todo value cannot be empty!";
  }
};
</script>

<template>
  <div class="input-wrap" :class="{ 'input-err': todo.invalid }">
    <input
      type="text"
      v-model="todo.todo"
      @keyup.enter="createTodo"
      aria-label="New Todo"
    />
    <button @click="createTodo">
      <slot name="button-content">{{ todo.buttonText }}</slot>
    </button>
  </div>
  <p class="filter-msg" v-if="todo.filterMsg">{{ todo.filterMsg }}</p>
  <p class="err-msg" v-if="todo.invalid">{{ todo.errMsg }}</p>
</template>

<style lang="scss" scoped>
.input-wrap {
  display: flex;
  transition: 250ms ease;
  border: 2px solid #41b080;

  &.input-err {
    border-color: red;
  }

  &:focus-within {
    box-shadow: 0 -4px 6px -1px rgb(0 0 0 / 0.1),
      0 -2px 4px -2px rgb(0 0 0 / 0.1);
  }

  input {
    width: 100%;
    padding: 8px 6px;
    border: none;

    &:focus {
      outline: none;
    }
  }

  button {
    padding: 8px 16px;
    border: none;
  }
}

.filter-msg,
.err-msg {
  margin-top: 6px;
  font-size: 12px;
  text-align: center;
}

.filter-msg {
  color: green;
}

.err-msg {
  color: red;
}
</style>

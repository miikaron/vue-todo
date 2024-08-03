<script setup>
import { Icon } from "@iconify/vue";
import { uid } from "uid";
import { ref, computed } from "vue";
import TodoCreator from "../components/TodoCreator.vue";
import TodoItem from "../components/TodoItem.vue";

const todoList = ref([]);
const filterText = ref(""); // Add a reactive property for filter text

// Computed property to check if all todos are completed
const todosCompleted = computed(() =>
  todoList.value.every((todo) => todo.isCompleted)
);

// Computed property to filter todos based on filterText
const filteredTodos = computed(() => {
  if (filterText.value.trim() === "") return todoList.value;
  return todoList.value.filter((todo) =>
    todo.todo.toLowerCase().includes(filterText.value.toLowerCase())
  );
});

// Fetch the todo list from local storage
const fetchTodoList = () => {
  try {
    const savedTodoList = JSON.parse(localStorage.getItem("todoList"));
    if (Array.isArray(savedTodoList)) {
      todoList.value = savedTodoList;
    }
  } catch (error) {
    console.error("Error parsing todo list from local storage", error);
  }
};

// Fetch todos on page load
fetchTodoList();

// Update local storage with the current todo list
const setTodoListLocalStorage = () => {
  localStorage.setItem("todoList", JSON.stringify(todoList.value));
};

// Create a new todo item
const createTodo = (todo) => {
  todoList.value.push({
    id: uid(),
    todo,
    isCompleted: false,
    isEditing: null,
  });
  setTodoListLocalStorage();
};

// Toggle the editing state of a todo item
const toggleEditTodo = (index) => {
  todoList.value[index].isEditing = !todoList.value[index].isEditing;
  setTodoListLocalStorage();
};

// Update the value of a todo item
const updateTodo = (todoVal, index) => {
  todoList.value[index].todo = todoVal;
  setTodoListLocalStorage();
};

// Toggle the completion state of a todo item
const toggleTodoComplete = (index) => {
  todoList.value[index].isCompleted = !todoList.value[index].isCompleted;
  setTodoListLocalStorage();
};

// Delete a todo item
const deleteTodo = (todo) => {
  todoList.value = todoList.value.filter(
    (todoFilter) => todoFilter.id !== todo.id
  );
  setTodoListLocalStorage();
};

// Update filter text based on input from TodoCreator
const updateFilterText = (text) => {
  filterText.value = text;
};
</script>

<template>
  <main>
    <h1>Create Todo</h1>
    <TodoCreator
      @create-todo="createTodo"
      :todoList="todoList"
      @update-filter-text="updateFilterText"
    >
      <template #button-content></template>
    </TodoCreator>
    <ul class="todo-list" v-if="filteredTodos.length > 0">
      <TodoItem
        v-for="(todo, index) in filteredTodos"
        :key="todo.id"
        :todo="todo"
        :index="index"
        @edit-todo="toggleEditTodo"
        @update-todo="updateTodo"
        @toggle-complete="toggleTodoComplete"
        @delete-todo="deleteTodo"
      />
    </ul>
    <p v-else class="todos-msg">
      <Icon icon="noto-v1:sad-but-relieved-face" />
      <span>You have no todos to complete</span>
    </p>
    <p v-if="todosCompleted && todoList.length > 0" class="todos-msg">
      <Icon icon="noto-v1:party-popper" />
      <span>You have completed all your todos!</span>
    </p>
  </main>
</template>

<style scoped lang="scss">
main {
  display: flex;
  flex-direction: column;
  max-width: 500px;
  width: 100%;
  margin: 0 auto;
  padding: 40px 16px;

  h1 {
    margin-bottom: 16px;
    text-align: center;
  }

  .todo-list {
    display: flex;
    flex-direction: column;
    list-style: none;
    margin-top: 24px;
    gap: 20px;
  }

  .todos-msg {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    margin-top: 24px;
  }
}
</style>

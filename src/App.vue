<template>
  <section class="todoapp">
    <Header @submit="addTodo" />
    <section v-if="todos.length !== 0" class="main">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="allCompleted"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list" v-for="todo in todos" v-bind:key="todo.key">
        <li
          :class="{
            completed: todo.completed,
            editing: editingTodoId === todo.id,
          }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" />
            <label @dblclick="setEditing(todo)">
              {{ todo.text }}
            </label>
            <button @click="deleteTodo(todo)" class="destroy"></button>
          </div>
          <input
            class="edit"
            v-model="todo.text"
            @blur="exitEditing(todo)"
            @keydown.enter="commitEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer v-if="todos.length !== 0" class="footer">
      <span class="todo-count">
        <strong>{{ activeItems.length }}</strong>
        {{ pluralize("item", activeItems.length) }} left
      </span>
      <ul class="filters">
        <li>
          <a class="selected" href="#/">All</a>
        </li>
        <li>
          <a href="#/active">Active</a>
        </li>
        <li>
          <a href="#/completed">Completed</a>
        </li>
      </ul>
      <button v-if="completedItems.length !== 0" class="clear-completed">
        Clear completed
      </button>
    </footer>
  </section>
</template>

<script>
import "todomvc-common/base.css";
import "todomvc-app-css/index.css";
import { computed, ref } from "vue";
import Header from "./components/Header.vue";

export default {
  components: {
    Header,
  },
  setup() {
    const todos = ref([]);
    const id = ref(0);
    const filter = ref("ALL");
    const editingTodoId = ref(null);
    const editingTodoValueBackup = ref("");

    const completedItems = computed(() =>
      todos.value.filter((todo) => todo.completed)
    );

    const activeItems = computed(() =>
      todos.value.filter((todo) => !todo.completed)
    );

    const allCompleted = computed({
      get: () => activeItems.value.length === 0,
      set: (value) => {
        for (const todo of todos.value) {
          todo.completed = value;
        }
      },
    });

    function addTodo(text) {
      todos.value.push({
        text,
        completed: false,
        id: id.value++,
      });
    }

    function deleteTodo(target) {
      todos.value = todos.value.filter((todo) => todo.id !== target.id);
    }

    function pluralize(word, quantity) {
      return quantity === 1 ? word : word + "s";
    }

    function setEditing(todo) {
      editingTodoValueBackup.value = todo.text;
      editingTodoId.value = todo.id;
    }

    function exitEditing(todo) {
      todo.text = editingTodoValueBackup.value;
      editingTodoId.value = null;
      editingTodoValueBackup.value = null;
    }

    function commitEdit() {
      editingTodoId.value = null;
    }

    return {
      todos,
      filter,
      completedItems,
      activeItems,
      allCompleted,

      addTodo,
      deleteTodo,
      pluralize,
      editingTodoId,
      setEditing,
      exitEditing,
      commitEdit,
      editingTodoValueBackup,
    };
  },
};
</script>

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
            v-focus="todo.id === editingTodoId"
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
      <button
        @click="clearCompleted"
        v-if="completedItems.length !== 0"
        class="clear-completed"
      >
        Clear completed
      </button>
    </footer>
  </section>
</template>

<script>
import "todomvc-common/base.css";
import "todomvc-app-css/index.css";
import { computed, ref, watch, watchEffect } from "vue";
import Header from "./components/Header.vue";

export default {
  components: {
    Header,
  },
  directives: {
    focus: {
      mounted(el) {
        el.focus();
      },
    },
  },
  setup() {
    const STORAGE_NAMESPACE = "todos";

    function deserialize() {
      const storage = localStorage.getItem(STORAGE_NAMESPACE);
      if (storage == null) {
        return undefined;
      }

      // TODO catch err
      const todos = JSON.parse(storage);

      return todos.map((todo, index) => ({ ...todo, id: index }));
    }
    const todos = ref(deserialize() ?? []);

    watchEffect(() => {
      localStorage.setItem(STORAGE_NAMESPACE, JSON.stringify(todos.value));
    });

    const id = ref(todos.value.length);
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

    function clearCompleted() {
      todos.value = activeItems.value;
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
      clearCompleted,
    };
  },
};
</script>

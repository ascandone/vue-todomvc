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
      <ul class="todo-list" v-for="todo in visibleTodos" v-bind:key="todo.key">
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
            v-focus="editingTodoId === todo.id"
            class="edit"
            v-model="todo.text"
            @blur="commitEdit(todo)"
            @keyup.enter="commitEdit(todo)"
            @keyup.esc="exitEditing(todo)"
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
          <a :class="{ selected: filter === 'ALL' }" href="#/">All</a>
        </li>
        <li>
          <a :class="{ selected: filter === 'ACTIVE' }" href="#/active">
            Active
          </a>
        </li>
        <li>
          <a :class="{ selected: filter === 'COMPLETED' }" href="#/completed">
            Completed
          </a>
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
      updated(el, binding) {
        if (binding.value) {
          el.focus();
        }
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
    window.addEventListener("hashchange", () => {
      switch (location.hash) {
        case "#/active":
          filter.value = "ACTIVE";
          break;
        case "#/completed":
          filter.value = "COMPLETED";
          break;
        default:
          filter.value = "ALL";
      }
    });

    const visibleTodos = computed(() => {
      switch (filter.value) {
        case "ALL":
          return todos.value;
        case "COMPLETED":
          return completedItems.value;
        case "ACTIVE":
          return activeItems.value;
      }
    });

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

    return {
      todos,
      filter,
      completedItems,
      activeItems,
      allCompleted,
      editingTodoId,
      editingTodoValueBackup,
      visibleTodos,
      addTodo(text) {
        todos.value.push({
          text,
          completed: false,
          id: id.value++,
        });
      },
      deleteTodo(target) {
        todos.value = todos.value.filter((todo) => todo.id !== target.id);
      },
      pluralize(word, quantity) {
        return quantity === 1 ? word : word + "s";
      },
      setEditing(todo) {
        editingTodoValueBackup.value = todo.text;
        editingTodoId.value = todo.id;
      },
      exitEditing(todo) {
        todo.text = editingTodoValueBackup.value;
        editingTodoId.value = null;
        editingTodoValueBackup.value = null;
      },
      commitEdit() {
        editingTodoId.value = null;
      },
      clearCompleted() {
        todos.value = activeItems.value;
      },
    };
  },
};
</script>

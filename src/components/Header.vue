<template>
  <header class="header">
    <h1>todos</h1>
    <input
      @keydown.enter="onPressedEnter"
      v-model="value"
      class="new-todo"
      placeholder="What needs to be done?"
      autofocus
    />
  </header>
</template>

<script>
import { ref } from "vue";

export default {
  setup(_, ctx) {
    const value = ref("");

    function onPressedEnter() {
      if (value.value.trim() !== "") {
        ctx.emit("submit", value.value);
        value.value = "";
      }
    }

    return {
      value,
      onPressedEnter,
    };
  },
};
</script>
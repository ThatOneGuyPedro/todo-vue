<template>
  <form @submit.prevent="onSubmit">
    <h2 class="label-wrapper">
      <label for="new-todo-input" class="label__lg">
        What needs to be done?
      </label>
    </h2>
    <input
      type="text"
      id="new-todo-input"
      name="new-todo"
      autocomplete="off"
      v-model.lazy.trim="label"
      class="input__lg"
      placeholder="e.g., create todo list" />
    <div class="duration-wrapper">
      <label for="duration-input" class="duration-label">
        Duration (minutes, optional):
      </label>
      <input
        type="number"
        id="duration-input"
        name="duration"
        min="1"
        max="999"
        v-model.number="duration"
        class="duration-input"
        placeholder="e.g., 25" />
    </div>
    <button type="submit" class="btn btn__primary btn__lg">Add</button>
  </form>
</template>

<script>
export default {
  methods: {
    onSubmit() {
      if (this.label === "") {
        return;
      }
      const todoData = {
        label: this.label,
        duration: this.duration || null,
      };
      this.$emit("todo-added", todoData);
      this.label = "";
      this.duration = null;
    },
  },
  data() {
    return {
      label: "",
      duration: null,
    };
  },
};
</script>

<style scoped>
.duration-wrapper {
  margin-bottom: 1rem;
}

.duration-label {
  display: block;
  font-size: 1.2rem;
  margin-bottom: 0.5rem;
  color: #4d4d4d;
}

.duration-input {
  width: 100%;
  padding: 0.5rem;
  border: 2px solid #4d4d4d;
  font-size: 1.2rem;
  box-sizing: border-box;
}

.duration-input:focus {
  border-color: #000;
  outline: 3px solid #228bec;
}
</style>

<template>
  <div class="stack-small" v-if="!isEditing">
    <div class="custom-checkbox">
      <input
        type="checkbox"
        class="checkbox"
        :id="id"
        :checked="isDone"
        @change="$emit('checkbox-changed')" />
      <label :for="id" class="checkbox-label">
        {{ label }}
        <span v-if="duration" class="duration-badge">{{ duration }} min</span>
      </label>
    </div>

    <div v-if="duration" class="timer-section" :class="{ 'timer-running': timerRunning }">
      <div class="timer-display">
        <span class="timer-icon">⏱️</span>
        <span class="timer-text">{{ formattedTimeRemaining }}</span>
      </div>
      <div class="timer-controls">
        <button
          v-if="!timerRunning"
          type="button"
          class="btn btn-timer btn-start"
          @click="startTimer">
          Start
        </button>
        <button
          v-else
          type="button"
          class="btn btn-timer btn-pause"
          @click="pauseTimer">
          Pause
        </button>
        <button
          type="button"
          class="btn btn-timer btn-reset"
          @click="resetTimer">
          Reset
        </button>
      </div>
    </div>

    <div class="btn-group">
      <button
        type="button"
        class="btn"
        ref="editButton"
        @click="toggleToItemEditForm">
        Edit
        <span class="visually-hidden">{{ label }}</span>
      </button>
      <button type="button" class="btn btn__danger" @click="deleteToDo">
        Delete
        <span class="visually-hidden">{{ label }}</span>
      </button>
    </div>
  </div>
  <to-do-item-edit-form
    v-else
    :id="id"
    :label="label"
    @item-edited="itemEdited"
    @edit-cancelled="editCancelled"></to-do-item-edit-form>
</template>

<script>
import ToDoItemEditForm from "./ToDoItemEditForm.vue";

export default {
  components: {
    ToDoItemEditForm,
  },
  props: {
    label: { required: true, type: String },
    done: { default: false, type: Boolean },
    id: { required: true, type: String },
    duration: { default: null, type: Number },
    timeRemaining: { default: null, type: Number },
    timerRunning: { default: false, type: Boolean },
  },
  data() {
    return {
      isEditing: false,
    };
  },
  computed: {
    isDone() {
      return this.done;
    },
    formattedTimeRemaining() {
      if (this.timeRemaining === null) return null;
      const minutes = Math.floor(this.timeRemaining / 60);
      const seconds = this.timeRemaining % 60;
      return `${minutes}:${seconds.toString().padStart(2, '0')}`;
    },
  },
  methods: {
    deleteToDo() {
      this.$emit("item-deleted");
    },
    toggleToItemEditForm() {
      console.log(this.$refs.editButton);
      this.isEditing = true;
    },
    itemEdited(newLabel) {
      this.$emit("item-edited", newLabel);
      this.isEditing = false;
      this.focusOnEditButton();
    },
    editCancelled() {
      this.isEditing = false;
      this.focusOnEditButton();
    },
    focusOnEditButton() {
      this.$nextTick(() => {
        const editButtonRef = this.$refs.editButton;
        editButtonRef.focus();
      });
    },
    startTimer() {
      this.$emit("timer-started");
    },
    pauseTimer() {
      this.$emit("timer-paused");
    },
    resetTimer() {
      this.$emit("timer-reset");
    },
  },
};
</script>

<style scoped>
.duration-badge {
  display: inline-block;
  margin-left: 0.5rem;
  padding: 0.2rem 0.5rem;
  background-color: #228bec;
  color: white;
  border-radius: 0.3rem;
  font-size: 0.85em;
  font-weight: bold;
}

.timer-section {
  background-color: #f5f5f5;
  border: 2px solid #ddd;
  border-radius: 0.5rem;
  padding: 1rem;
  margin: 1rem 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
  transition: all 0.3s ease;
}

.timer-section.timer-running {
  background-color: #e8f5e9;
  border-color: #4caf50;
}

.timer-display {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1.8rem;
  font-weight: bold;
}

.timer-icon {
  font-size: 2rem;
}

.timer-text {
  font-family: monospace;
  color: #333;
}

.timer-running .timer-text {
  color: #2e7d32;
}

.timer-controls {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.btn-timer {
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
  min-width: 4.5rem;
}

.btn-start {
  background-color: #4caf50;
  color: white;
  border-color: #45a049;
}

.btn-start:hover {
  background-color: #45a049;
}

.btn-pause {
  background-color: #ff9800;
  color: white;
  border-color: #e68900;
}

.btn-pause:hover {
  background-color: #e68900;
}

.btn-reset {
  background-color: #757575;
  color: white;
  border-color: #616161;
}

.btn-reset:hover {
  background-color: #616161;
}

.custom-checkbox > .checkbox-label {
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-weight: 400;
  font-size: 16px;
  font-size: 1rem;
  line-height: 1.25;
  color: #0b0c0c;
  display: block;
  margin-bottom: 5px;
}
.custom-checkbox > .checkbox {
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-weight: 400;
  font-size: 16px;
  font-size: 1rem;
  line-height: 1.25;
  box-sizing: border-box;
  width: 100%;
  height: 40px;
  height: 2.5rem;
  margin-top: 0;
  padding: 5px;
  border: 2px solid #0b0c0c;
  border-radius: 0;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}
.custom-checkbox > input:focus {
  outline: 3px dashed #fd0;
  outline-offset: 0;
  box-shadow: inset 0 0 0 2px;
}
.custom-checkbox {
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  font-weight: 400;
  font-size: 1.6rem;
  line-height: 1.25;
  display: block;
  position: relative;
  min-height: 40px;
  margin-bottom: 10px;
  padding-left: 40px;
  clear: left;
}
.custom-checkbox > input[type="checkbox"] {
  -webkit-font-smoothing: antialiased;
  cursor: pointer;
  position: absolute;
  z-index: 1;
  top: -2px;
  left: -2px;
  width: 44px;
  height: 44px;
  margin: 0;
  opacity: 0;
}
.custom-checkbox > .checkbox-label {
  font-size: inherit;
  font-family: inherit;
  line-height: inherit;
  display: inline-block;
  margin-bottom: 0;
  padding: 8px 15px 5px;
  cursor: pointer;
  touch-action: manipulation;
}
.custom-checkbox > label::before {
  content: "";
  box-sizing: border-box;
  position: absolute;
  top: 0;
  left: 0;
  width: 40px;
  height: 40px;
  border: 2px solid currentColor;
  background: transparent;
}
.custom-checkbox > input[type="checkbox"]:focus + label::before {
  border-width: 4px;
  outline: 3px dashed #228bec;
}
.custom-checkbox > label::after {
  box-sizing: content-box;
  content: "";
  position: absolute;
  top: 11px;
  left: 9px;
  width: 18px;
  height: 7px;
  transform: rotate(-45deg);
  border: solid;
  border-width: 0 0 5px 5px;
  border-top-color: transparent;
  opacity: 0;
  background: transparent;
}
.custom-checkbox > input[type="checkbox"]:checked + label::after {
  opacity: 1;
}
@media only screen and (min-width: 40rem) {
  label,
  input,
  .custom-checkbox {
    font-size: 19px;
    font-size: 1.9rem;
    line-height: 1.31579;
  }
}
</style>

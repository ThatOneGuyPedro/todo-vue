<template>
  <div id="app">
    <h1>To-Do List</h1>
    <to-do-form @todo-added="addToDo"></to-do-form>
    <h2 id="list-summary" ref="listSummary" tabindex="-1">{{ listSummary }}</h2>
    <ul aria-labelledby="list-summary" class="stack-large">
      <li v-for="item in ToDoItems" :key="item.id">
        <to-do-item
          :label="item.label"
          :done="item.done"
          :id="item.id"
          :duration="item.duration"
          :timeRemaining="item.timeRemaining"
          :timerRunning="item.timerRunning"
          @checkbox-changed="updateDoneStatus(item.id)"
          @item-deleted="deleteToDo(item.id)"
          @item-edited="editToDo(item.id, $event)"
          @timer-started="startTimer(item.id)"
          @timer-paused="pauseTimer(item.id)"
          @timer-reset="resetTimer(item.id)">
        </to-do-item>
      </li>
    </ul>
    <toast-notification
      v-for="(notification, index) in notifications"
      :key="index"
      :message="notification.message"
      :type="notification.type"
      :duration="notification.duration"
      @close="removeNotification(index)"
      :style="{ top: `${2 + index * 6}rem` }">
    </toast-notification>
  </div>
</template>

<script>
import ToDoItem from "./components/ToDoItem.vue";
import ToDoForm from "./components/ToDoForm.vue";
import ToastNotification from "./components/ToastNotification.vue";
import { nanoid } from "nanoid";

export default {
  name: "app",
  components: {
    ToDoItem,
    ToDoForm,
    ToastNotification,
  },
  data() {
    return {
      ToDoItems: [],
      notifications: [],
    };
  },
  mounted() {
    this.loadFromLocalStorage();
  },
  methods: {
    addToDo(toDoData) {
      this.ToDoItems.push({
        id: "todo-" + nanoid(),
        label: typeof toDoData === 'string' ? toDoData : toDoData.label,
        done: false,
        duration: typeof toDoData === 'object' ? toDoData.duration : null,
        timeRemaining: typeof toDoData === 'object' && toDoData.duration ? toDoData.duration * 60 : null,
        timerRunning: false,
      });
      this.saveToLocalStorage();
    },
    updateDoneStatus(toDoId) {
      const toDoToUpdate = this.ToDoItems.find((item) => item.id === toDoId);
      toDoToUpdate.done = !toDoToUpdate.done;
      this.saveToLocalStorage();
    },
    deleteToDo(toDoId) {
      const itemIndex = this.ToDoItems.findIndex((item) => item.id === toDoId);
      this.ToDoItems.splice(itemIndex, 1);
      this.$refs.listSummary.focus();
      this.saveToLocalStorage();
    },
    editToDo(toDoId, newLabel) {
      const toDoToEdit = this.ToDoItems.find((item) => item.id === toDoId);
      toDoToEdit.label = newLabel;
      this.saveToLocalStorage();
    },
    startTimer(toDoId) {
      const item = this.ToDoItems.find((item) => item.id === toDoId);
      if (!item || !item.duration || item.timerRunning) return;

      item.timerRunning = true;
      item.timerInterval = setInterval(() => {
        if (item.timeRemaining > 0) {
          item.timeRemaining--;
          this.saveToLocalStorage();
        } else {
          this.pauseTimer(toDoId);
          this.showNotification(`Time's up for: ${item.label}`, 'timer', 6000);
        }
      }, 1000);
    },
    pauseTimer(toDoId) {
      const item = this.ToDoItems.find((item) => item.id === toDoId);
      if (!item) return;

      item.timerRunning = false;
      if (item.timerInterval) {
        clearInterval(item.timerInterval);
        item.timerInterval = null;
      }
      this.saveToLocalStorage();
    },
    resetTimer(toDoId) {
      const item = this.ToDoItems.find((item) => item.id === toDoId);
      if (!item || !item.duration) return;

      this.pauseTimer(toDoId);
      item.timeRemaining = item.duration * 60;
      this.saveToLocalStorage();
    },
    showNotification(message, type = 'success', duration = 4000) {
      this.notifications.push({ message, type, duration });
    },
    removeNotification(index) {
      this.notifications.splice(index, 1);
    },
    saveToLocalStorage() {
      const itemsToSave = this.ToDoItems.map(item => ({
        id: item.id,
        label: item.label,
        done: item.done,
        duration: item.duration,
        timeRemaining: item.timeRemaining,
      }));
      localStorage.setItem('todoItems', JSON.stringify(itemsToSave));
    },
    loadFromLocalStorage() {
      const saved = localStorage.getItem('todoItems');
      if (saved) {
        try {
          const items = JSON.parse(saved);
          this.ToDoItems = items.map(item => ({
            ...item,
            timerRunning: false,
            timerInterval: null,
          }));
        } catch (e) {
          console.error('Failed to load todos from localStorage:', e);
        }
      }
    },
  },
  beforeUnmount() {
    this.ToDoItems.forEach(item => {
      if (item.timerInterval) {
        clearInterval(item.timerInterval);
      }
    });
  },
  computed: {
    listSummary() {
      const numberFinishedItems = this.ToDoItems.filter(
        (item) => item.done,
      ).length;
      return `${numberFinishedItems} out of ${this.ToDoItems.length} items completed`;
    },
  },
};
</script>

<style>
/* Global styles */
.btn {
  padding: 0.8rem 1rem 0.7rem;
  border: 0.2rem solid #4d4d4d;
  cursor: pointer;
  text-transform: capitalize;
}
.btn__danger {
  color: #fff;
  background-color: #ca3c3c;
  border-color: #bd2130;
}
.btn__filter {
  border-color: lightgrey;
}
.btn__danger:focus {
  outline-color: #c82333;
}
.btn__primary {
  color: #fff;
  background-color: #000;
}
.btn-group {
  display: flex;
  justify-content: space-between;
}
.btn-group > * {
  flex: 1 1 auto;
}
.btn-group > * + * {
  margin-left: 0.8rem;
}
.label-wrapper {
  margin: 0;
  flex: 0 0 100%;
  text-align: center;
}
[class*="__lg"] {
  display: inline-block;
  width: 100%;
  font-size: 1.9rem;
}
[class*="__lg"]:not(:last-child) {
  margin-bottom: 1rem;
}
@media screen and (min-width: 620px) {
  [class*="__lg"] {
    font-size: 2.4rem;
  }
}
.visually-hidden {
  position: absolute;
  height: 1px;
  width: 1px;
  overflow: hidden;
  clip: rect(1px 1px 1px 1px);
  clip: rect(1px, 1px, 1px, 1px);
  clip-path: rect(1px, 1px, 1px, 1px);
  white-space: nowrap;
}
[class*="stack"] > * {
  margin-top: 0;
  margin-bottom: 0;
}
.stack-small > * + * {
  margin-top: 1.25rem;
}
.stack-large > * + * {
  margin-top: 2.5rem;
}
@media screen and (min-width: 550px) {
  .stack-small > * + * {
    margin-top: 1.4rem;
  }
  .stack-large > * + * {
    margin-top: 2.8rem;
  }
}
/* End global styles */
#app {
  background: #fff;
  margin: 2rem 0 4rem 0;
  padding: 1rem;
  padding-top: 0;
  position: relative;
  box-shadow:
    0 2px 4px 0 rgba(0, 0, 0, 0.2),
    0 2.5rem 5rem 0 rgba(0, 0, 0, 0.1);
}
@media screen and (min-width: 550px) {
  #app {
    padding: 4rem;
  }
}
#app > * {
  max-width: 50rem;
  margin-left: auto;
  margin-right: auto;
}
#app > form {
  max-width: 100%;
}
#app h1 {
  display: block;
  min-width: 100%;
  width: 100%;
  text-align: center;
  margin: 0;
  margin-bottom: 1rem;
}
</style>

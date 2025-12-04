<template>
  <transition name="toast">
    <div v-if="visible" class="toast-notification" :class="type">
      <div class="toast-content">
        <span class="toast-icon">{{ icon }}</span>
        <span class="toast-message">{{ message }}</span>
      </div>
      <button class="toast-close" @click="close">×</button>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    message: { type: String, required: true },
    type: { type: String, default: 'success' },
    duration: { type: Number, default: 4000 },
  },
  data() {
    return {
      visible: false,
    };
  },
  computed: {
    icon() {
      switch (this.type) {
        case 'success':
          return '✓';
        case 'warning':
          return '⚠';
        case 'error':
          return '✕';
        case 'timer':
          return '⏰';
        default:
          return 'ℹ';
      }
    },
  },
  mounted() {
    this.visible = true;
    if (this.duration > 0) {
      setTimeout(() => {
        this.close();
      }, this.duration);
    }
  },
  methods: {
    close() {
      this.visible = false;
      setTimeout(() => {
        this.$emit('close');
      }, 300);
    },
  },
};
</script>

<style scoped>
.toast-notification {
  position: fixed;
  top: 2rem;
  right: 2rem;
  min-width: 300px;
  max-width: 500px;
  padding: 1rem 1.5rem;
  background-color: white;
  border-radius: 0.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  display: flex;
  align-items: center;
  justify-content: space-between;
  z-index: 1000;
  border-left: 4px solid;
}

.toast-notification.success {
  border-left-color: #4caf50;
}

.toast-notification.warning {
  border-left-color: #ff9800;
}

.toast-notification.error {
  border-left-color: #f44336;
}

.toast-notification.timer {
  border-left-color: #2196f3;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.toast-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.toast-icon {
  font-size: 1.5rem;
  font-weight: bold;
}

.toast-message {
  font-size: 1rem;
  line-height: 1.4;
}

.toast-close {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0;
  margin-left: 1rem;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.toast-notification.timer .toast-close {
  color: white;
}

.toast-close:hover {
  opacity: 1;
}

.toast-enter-active,
.toast-leave-active {
  transition: all 0.3s ease;
}

.toast-enter-from {
  transform: translateX(100%);
  opacity: 0;
}

.toast-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}

@media screen and (max-width: 550px) {
  .toast-notification {
    top: 1rem;
    right: 1rem;
    left: 1rem;
    min-width: auto;
  }
}
</style>

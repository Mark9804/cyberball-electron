<template>
  <div class="button" :class="getButtonType" role="button">{{ text }}</div>
</template>

<script>
export default {
  name: "startButton",
  props: {
    text: String,
    type: {
      type: String,
      default: "confirm",
    },
  },
  computed: {
    getButtonType() {
      switch (this.type) {
        case "confirm":
          return "";
        case "danger":
          return "warning";
        case "disabled":
          return "disabled";
        default:
          return "";
      }
    },
  },
};
</script>

<style scoped>
.button {
  font-size: 1.5rem;
  font-weight: bold;
  padding: 1rem;
  border-radius: 10px;
  background: #4fc08d;
  color: #fff;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  margin-top: 1rem;
  user-select: none;
  min-width: 20vw;
}

.warning {
  background: #ff230d;
}

.disabled {
  background: rgba(204, 204, 204, 0.99);
  color: #666;
  cursor: not-allowed;
}

.disabled:hover {
  box-shadow: none !important;
}

.button:hover {
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}
</style>

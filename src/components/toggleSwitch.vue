<template>
  <input class="switch" type="checkbox" :checked="checked" @change="toggle" />
</template>

<script>
export default {
  name: "toggleSwitch",
  data() {
    return {
      checked: this.isChecked,
    };
  },
  props: {
    isChecked: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    toggle() {
      this.checked = !this.checked;
      this.$emit("updateClickValue", this.checked);
    },
  },
};
</script>

<style scoped>
.switch {
  appearance: none;
  border: none !important;
  outline: none !important;
  min-width: 3rem;
  max-width: 3rem;
  height: 1.15rem;
  border-radius: 1rem;
  box-sizing: content-box;
  display: grid;
  grid-template-areas: "track";
  grid-template-columns: 1fr;
  justify-items: start;
  align-items: center;
  background-color: #ff230d;
  transition: all 0.2s ease-in-out;
  cursor: pointer;
}

input::before,
input:checked::before {
  grid-area: track;
  justify-self: center;
  color: #fff;
  transform: translateY(-0.05rem);
}

input::before {
  content: "|";
  font-size: 0.6rem;
  -webkit-text-stroke: 1px;
}

input:checked::before {
  content: "◯";
  font-size: 0.6rem;
  -webkit-text-stroke: 1px;
}

/* button-thumb */
input::after {
  position: relative;
  grid-area: track;
  content: "";
  width: 1.25rem;
  height: 1.25rem;
  border-radius: 50%;
  background: linear-gradient(145deg, #dadada, #ffffff);
  transition: all 0.2s ease-in-out;
  box-shadow: 0 0 6px #666666;
  transform: translate(-0.2rem, -0.05rem);
}

input:checked::after {
  position: relative;
  grid-area: track;
  transform: translate(2.2rem, -0.05rem);
}

input:checked {
  background-color: #4fc08d;
}

input:checked {
}
</style>

<template>
  <ul>
    <li
      v-for="language in availableLang"
      :key="language.locale"
      @click="updateLang(language)"
    >
      {{ language.name }}
    </li>
  </ul>
</template>

<script>
export default {
  name: "langSelectionMenu",
  props: {
    availableLang: Array,
  },
  emits: ["updateLang"],
  data() {
    return {
      selectedLang: "",
    };
  },
  setup(props, context) {
    const updateLang = (lang) => {
      context.emit("updateLang", lang);
    };

    return { updateLang };
  },
};
</script>

<style scoped>
li {
  user-select: none;
  cursor: pointer;
  text-align: center;
  line-height: 2;
  transition: background-color 0.175s ease-in-out;
}

li:hover {
  background-color: #eeeeeee5;
}
</style>

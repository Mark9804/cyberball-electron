<template>
  <h1>{{ titleString }}</h1>
  <div v-html="briefingString"></div>
</template>

<script>
export default {
  name: "welcomeScreen",
  props: {
    briefingString: String,
    titleString: String,
  },
};
</script>

<style></style>

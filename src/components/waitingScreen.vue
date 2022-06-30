<template>
  <div class="container-center">
    <h1>
      {{ welcomeString
      }}{{ new RegExp("[A-Za-z]").test(welcomeString) ? ", " : "，"
      }}{{ participantName }}!
    </h1>
    <h2>
      {{ matchingString }}
      <span id="animation1">.</span>
      <span id="animation2">.</span>
      <span id="animation3">.</span>
    </h2>
  </div>
</template>

<script>
export default {
  name: "waitingScreen",
  props: {
    participantName: String,
    welcomeString: String,
    matchingString: String,
  },
};
</script>

<style scoped>
span {
  display: inline-block;
}

#animation1 {
  animation: jump 1s steps(3) infinite;
  animation-delay: 0s;
}

#animation2 {
  animation: jump 1s steps(3) infinite;
  animation-delay: 0.25s;
}

#animation3 {
  animation: jump 1s steps(3) infinite;
  animation-delay: 0.5s;
}

@keyframes jump {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
  100% {
    transform: translateY(0);
  }
}
</style>
